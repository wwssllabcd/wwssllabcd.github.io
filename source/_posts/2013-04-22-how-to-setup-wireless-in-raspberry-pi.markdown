---
layout: post
title: "Raspberry pi: 設定無線網路"
date: 2013-04-22 16:21
comments: true
tags: [How-To, Raspberry Pi]
---

安裝 wireless usb 
-----------------
經過我的實驗，UsbHub 對於無線傳輸會不會當機佔了很大的因素，我之前使用 D-link Dub H7 用samba拉檔案，約 5min 後就會當機，起初還以為RPi本身的問題，
後來換了Belkin F5U237之後，試了兩張網卡( SMC-usb-wireless-G，TL-821Nv3)都沒有問題。

最好是買有內建驅動不用再折騰的如 EW-7811Un、TL-821Nv3 等，
你也可以在[Wifi硬體清單](http://elinux.org/RPi_VerifiedPeripherals#USB_Wi-Fi_Adapters)中選一張。  

<!--more-->

安裝驅動程式  
===============
接下來進入主題，安裝驅動程式   
先把usb插上去之後，輸入`lsusb`，查看看型號為何，我這邊是以 SMC-usb-wireless-G 為例子，所以這邊查到的是 zd1211rw  

使用apt裝驅動程式

	sudo apt-get update
	
使用 apt-cache來查看有哪些驅動

	apt-cache search zd1211
	
顯示

    zd1211-firmware - Firmware images for the zd1211rw wireless driver

所以這邊知道要安裝 zd1211-firmware

	sudo apt-get install zd1211-firmware

使用`lsmod`，查看看 zd1211有沒有被載入，

	使用`ifconfig`，看看wlan0有沒有起來

呼叫wlan0掃瞄附近的 AP

	sudo iwlist wlan0 scan
	
或是

	sudo iwlist wlan0 scan | grep SSID

如果有掃瞄到，代表有正確的驅動起來，現在連上無線 AP

安裝 wpa client 驅動，預設已經安裝，若沒有可以輸入以下指令

	sudo apt-get install wpasupplicant

編輯 interfaces 

	sudo nano /etc/network/interfaces

加入以下設定
```
	auto lo
	iface lo inet loopback
	iface eth0 inet dhcp
	allow-hotplug wlan0
	iface wlan0 inet dhcp #使用dhcp配置

	#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	#iface default inet dhcp
	
	#======= new setting =======
	#auto eth0  #如果 eth0 抓不到的話， 可以試著把它打開
	#auto wlan0 #自動開啟wlan0
	wpa-conf /etc/wpa.conf  # wpa 檔案放在此
	
```

新增 wpa.conf 檔

	sudo nano /etc/wpa.conf
	
而 wpa.config 的內容如下，其中的SSID更換成要連的無線 AP SSID，而 psk 填入連線的密碼(ex:123456)
```
	network={
		ssid="MyAP"
		psk="12345678"
	}
```

不過這邊使用密碼是明碼不太好, 可以使用 wpa_passphrase 把密碼加密一下, wpa_passphrase的輸入格式如下  

	wpa_passphrase <ssid> [passphrase]
	
所以輸入 wpa_passphrase MyAP 12345678 就會出現  
```
	network={
		ssid="MyAp"
        #psk="12345678"
        psk=b2449175398db27f75a0790f780cdacd0cbf8529e9e29fa051bdf3248f1fd595
	}
```
再把他產生出來的 psk 這段貼入wpa.conf中就可以了, 若是想建立多個AP的密碼表，可以這樣做  

```
#asus rt-n18u
network={
	ssid="EricWangAp_CH"
    psk="12345678"
}

#asus 520gu
network={
        ssid="EricWangAp"
        psk="12345678"
}
```


停用/啟用網卡
	
	sudo ifdown wlan0
	sudo ifup wlan0

使用 `ifconfig` 觀察是否正常取得 ip  

Reference
==============
這邊也提供其他網頁的資料，供各位參考  
[命令列設置無線網路.](http://www.raspberrypi.com.tw/2152/setting-up-wifi-with-the-command-line/)   

