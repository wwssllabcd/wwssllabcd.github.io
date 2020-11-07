---
layout: post
title: "Raspberry Pi 安裝心得、教學、簡介"
date: 2013-01-31 17:06
comments: true
tags: [How-To, Raspberry Pi]
toc: true 
---

簡介
==========
[Raspberry Pi 2]on sale   

Raspberry Pi 介紹可見[RPI Wiki]，以下簡稱 RPi，
是一款跟名片差不多大小的小電腦，採用 ARM 11架構，CPU大約 700MHz，由於售價便宜，基本上就是一台小電腦，

目前比較廣泛的應用如  

* 當作 NAS 動物機：使用 `transmission-daemon`  
* 當作家庭檔案 Server：使用 `Samba Server`  
* 當作無線 AP：使用 `hostapd`  
* 當作多媒體播放機看PPS之類的：使用 `Raspbmc`  
* 教小朋友寫 python 程式：RPi 最主要的功能  

當然RPi不只如此，這邊有個[34 個使用 Raspberry Pi 的酷創意]也許你會有興趣

開始之前
---------------
台灣已經有代理商[台灣樹莓派]可以訂購了，一片雖然只要 1833 元，但是你還是需要加購一些商品，如SD卡等，
代理商也有賣一些周邊商品，價格也挺合理，各位可以在上面一次購足，而這邊有建議的[硬體相容性清單]，
可以參考看看，否則買到不相容的裝置就麻煩了

Raspberry pi 2 (簡稱 RPI 2)也已經開賣了，除了之外，也要介紹有另一家叫 [RS 台灣]，東西更便宜，大家可以多多比較   

方便大家以下為購買的 Link，有更便宜的地方也歡迎留言  
[RS 台灣 RPI 2](http://twcn.rs-online.com/web/p/processor-microcontroller-development-kits/832-6274/)  
[台灣樹莓派 RPI 2](http://www.raspberrypi.com.tw/purchase/)  

購買清單
-----------
這邊我列出一些我個人建議的購買清單  
* `SD卡`：最好是選 SanDisk 的，4G以上。  
* `電源`：如果你手邊有 5V，又超過 2000mA 的 +microUSB介面的電源就可以使用了，通常手機的旅充是這樣的東西。不過供電超過 1A 到 mini USB給 pi都沒用，那邊有放保險絲1A。   
* `外殼`：建議購買，如果你不怕跟其他裝置 short 到的話，另外一提的是如果你有 GPIO 的需求，最好找那種可以接線進去的，否則買殼也沒意義。  
* `鍵盤`：可以不用買，因為通常是用 [pietty] terminal 進去使用。  
* `無線網卡`：強力推薦 [EDIMAX 訊舟 EW-7822UAn]，不但在 windows、ubuntu、Raspberry pi、Banana Pi、Mac OS 上全部都可以驅動外，可說是一個打十個，
且傳輸速度比我其他張卡還來的快，真心推薦  
* `USB-HUB`：極重要，外接裝置(USB硬碟，無線網卡..etc)穩不穩就要靠他了，所以從 [USB-HUB 建議清單]中購買一個有外接電源的hub，
我用過不好的 USB-HUB，USB 外接硬碟與 USB 無線網卡都是忽好忽壞的狀況，跑沒 10 min 就會當機的那種，當下很難抓出問題在哪。  

當然啦，有錢就全部都買也是不反對的。  

若想要瞭解不同硬體版本間的差異，這篇對 RPi 的硬體有詳細的介紹    
[Raspberry Pi (樹莓派) Model B 各版本之間的差異]  

<!--more-->

安裝
===========

安裝 OS
--------------
要安裝OS，必須要有張SD卡，並且去[RPI OS 官方下載頁面]看看，裡面有很多選擇，不過官方推薦 [Raspbian](https://www.raspberrypi.org/downloads/) 這套，
[下載 Raspbian] 回來解開得到img檔，也需要去下載 [Win32DiskImager] 這套程式，將剛剛解出來的 img 還原到SD卡上。

等到寫完後，直接插入到 RPI 上就可以使用了，可說是很簡單。  

若還是不行，這邊提供幾個比較詳細的教學，可以交互參考看看。   
[RPI官方燒錄 SD 卡教學]  
[燒錄 Raspberry Pi 的 SD 卡]  
[Raspberry Pi：安裝OS、簡易設定]   
[Raspberry Pi Model B 512MB 開箱、安裝與應用]   

如果對各種 OS 的差異有興趣的話，以下這篇[安裝於 Raspberry Pi 中的作業系統概觀]，有針對不同的 image file 作一些介紹    
對 RPI OS 有興趣的話，這邊也有自製的 RPI OS 叫[Raspberry Pi 上自幹一個微小作業系統]可以參考看看，()應該很少人對 OS 有興趣吧 )   

進入 OS 之前
--------------
RPI 的顯示方式有兩種，使用 HDMI 接到顯示裝置可能會有相容性問題，很有可能無法顯示，而使用AV端子就保險的多，
不過我比較推薦使用 terminal 的方式，既不用買線材，也不用多準備一套鍵盤滑鼠，只要你把網路線插上去，
由無線 AP 得知現在配到的 IP，就可以用 pietty 之類的 ssh 軟體登入

HDMI 無法正常顯示
--------------
如果 HDMI 可以正常顯示的可以跳過這段

有時候顯示裝置老舊, 就必須要特別設定 HDMI 參數
首先找到SD卡根目錄下面的 config.txt，打開來, 開始設定參數(若找不到該參數, 則自己加上去即可)
設定
```
	sdtv_mode = 0
	sdtv_aspect = 3
	hdmi_group = 1
	hdmi_mode = 1
```

sdtv_aspect 代表你的螢幕比例, 定義如下

	sdtv_aspect=1  4:3
	sdtv_aspect=2  14:9
	sdtv_aspect=3  16:9

hdmi_group=1 及 hdmi_mode=1 是代表使用 CEA + VGA 模式, 通常設定完 HDMI 會以最低標準顯示，
這兩個參數模式的組合在[RPi_config.txt -- Video_mode_options] 有列出來，這裡不再解釋

接下來你可以讓 RPI 抓你的螢幕有哪些模式可以選  
輸入以下指令

	/opt/vc/bin/tvservice -d edid.dat
	/opt/vc/bin/edidparser edid.dat

edidparser 會列出建議的組合，照著設定就可以了

pietty 中文顯示
--------------
當 OS 裝好之後，就可以使用 terminal 的方式進入系統，使用 [Pietty]  透過 ssh 方式連線 Linux系統，中文字會變亂碼
解決方式為把字元編碼設為 UTF-8
![pietty_utf8]  

登入 OS 以及安裝套件
=========

第一次進入 OS
--------------

登入的預設帳號/密碼為  

	pi/raspberry

進去之後，可以先執行以下指令換密碼  

	passwd pi

換完密碼，也順便執行  

	sudo raspi-config

進行一些基本設定，基本上會建議改好幾個選項，分述如下  
* `expand_rootfs`: 如果你使用大於4G以上的 SD 卡， 建議選一下，這樣才可以讓SD卡使用到全部容量   
* `configure_keyboard`: 建議選 Generic 105-key (Intl) PC鍵盤    
* `change_locale`: 去掉 en.GB, 加入 zh_TW.UTF-8 與 en_us.UTF8   
* `change_timezone `: 可以選 etc -> GMT+8的地區   
* `memory_split`: 要配置多少記憶體給 GPU，如果你是 Raspbmc 之類的 OS，可以給多一點，這個部分也可以在 SD 卡中根目錄的 config.txt 的 gpu_mem 選項修改   

詳細的設定可以看    
[RPi_raspi-config]，[使用raspi-config工具配置樹莓派]  

設定完之後，也可以進行apt的更新，如  

	sudo apt-get update
	sudo apt-get upgrade

安裝 VNC server
--------------
使用 apt 安裝  

	sudo apt-get install tightvncserver
	
安裝完，鍵入  

	vncserver :1 -geometry 1280x800 -depth 16 -pixelformat rgb565
	
即代表開啟 VNC server，此時再用 VNC client連入即可

外接 2.5 吋 USB HDD 硬碟認不到
--------------
基本上直接接上去就可以了，如果還是認不到的話，就換條線試看看，有時候是線的品質太差所導致供電不足  
若還是不行，請把 SD 卡中的 config.txt 中，加入以下參數

	max_usb_current=1
	
這樣會讓 USB 電壓供電到 1200 mA，不過有沒有後遺症我是不知道，若怕的話，還是接個外接 Usb-Hub 比較保險。  
其他的參數請見 [RPiconfig]


讀取 NTFS 格式，使用 NTFS-3G 套件
--------------
若要讀取 NTFS 的資料，必須要安裝 NTFS-3G 這個套件，使用 apt 指令  

	sudo apt-get install ntfs-3g
	
使用 ntfs-3g 在傳輸的時候會有點小耗 cpu 資源，所以還是建議沒必要，還是使用 ext4 會比較好
	
格式化檔案系統為 ext4
--------------
記得要先把系統 umount 之後，使用  

	sudo fdisk -l 
	
查看目前硬碟狀況，如果不是 ext4 的格式的話，則必須要做刪除 partation 的動作，使用  

	sudo fdisk /dev/sda    # sda 代表整個硬碟, 而 sda1 代表第一個 partition
	
進入到 fdisk 之後，可以使用`p`來列印目前有多少 partation，使用`d`來刪除，使用`n`來新增，最後使用`w`來寫入 partation 的資料後，再使用   

	sudo fdisk -l 
	
來查看，應該就可以看到該硬碟轉換成 linux file system 的格式了，接下來使用  

	sudo mkfs.ext4 /dev/sda1
	
再把 sda1 格式化為 ext4, 若想要查看是否硬碟格式是否正確為 ext4 , 可使用  

	sudo parted
	
進入到 parted 後, 鍵入`print`, 即可查看目前硬碟的格式

若是有 GDT 的問題, 可以改用 parted 來做分割, 如  

	sudo parted /dev/sda

掛載外部 usb storage
--------------
先使用 `lsusb` 來觀察 usb device 有沒有正確連上 RPI

先在 /media 下面建立一個資料夾叫做myusb  

	sudo mkdir /media/myusb
	 
使用 mount 指令，這邊以 NTFS 的外部儲存媒體作為範例  

	sudo mount -t ntfs-3g /dev/sda1 /media/myusb
	
如果要掛載 ext4 ，可以使用                

	sudo mount -t ext4 /dev/sda1 /media/myusb
	
掛載後, 可以使用`df`指令來查看是否有掛載成功  

不過這種掛載是暫時性的, 每次開機都要掛載一次, 如果要開機自動掛載的話, 必須修改`/etc/fstab`這個檔案, 加入以下設定  

	/dev/sda1       /media/myusb    ext4    defaults          0       0
	
即可在開機的時候,自動掛載`sda1`到`myusb`這個資料夾上
而檢查有無掛載成功的方式可以輸入

	mount

這個指令會把目前掛載的裝置與掛載點列出來, 這樣就可以做檢查了

這邊也提供其他網頁的資料，供各位參考。  
[How to mount and use a USB hard disk with the Raspberry Pi]  
[Force your Raspberry Pi to mount an external USB drive every time it starts up]  

安裝 RasPBX
--------------
安裝 RasPBX，在 RPI 上跑的 asterisk (網路電話)  

可參考[Asterisk for RPI 官方網頁]，下載他們專屬的[Asterisk OS image file]安裝，
安裝說明在該網站的[Document for Asterisk]中，由於這部份要整個替換掉系統，所以我就沒試了，有興趣的可以自行嘗試
網路上也有不少的中文資源如下  
[Asterisk -【Raspberry Pi】— 攻略]  

如果要安裝在 官方image上面的好像也是可以，raspbx 論壇有一篇有提供說明  
[Use raspbx repo on a Rasbian Official image ]  
[What is the difference between the RasPBX image and the Pi Store edition?]  

使用 Pi Store 也可以找到 asterisk 的套件，不過要安裝很多東西，如 Asterisk, FreePBX and Apache/PHP/Mysql  等重量級套件，
基於我已經把他變成BT機了，目前沒有研究的打算  

安裝 transmission-daemon 
--------------
請見[打造脫機下載農場，使用Raspberry Pi + Transmission-daemon]

設定無線網路
--------------
請見[Raspberry Pi: 設定無線網路]

安裝 Samba (網路芳鄰)
--------------
請見[Raspberry Pi 上安裝 Samba]

安裝 Raspbmc
--------------
請見 [Raspberry Pi 變成多媒體播放機 -- 安裝 Raspbmc]


[Raspberry Pi 上安裝 Samba]: http://wwssllabcd.github.io/blog/2013/04/22/how-to-setup-samba-in-raspberry-pi/
[打造脫機下載農場，使用Raspberry Pi + Transmission-daemon]: http://wwssllabcd.github.io/blog/2013/04/22/how-to-setup-transmission-deamon-in-raspberry-pi/
[Raspberry Pi: 設定無線網路]: http://wwssllabcd.github.io/blog/2013/04/22/how-to-setup-wireless-in-raspberry-pi/
[Raspberry Pi 變成多媒體播放機 -- 安裝 Raspbmc]: http://wwssllabcd.github.io/blog/2013/04/22/how-to-setup-raspbmc-in-raspberry-pi/

[Raspberry Pi 2]:http://www.raspberrypi.org/raspberry-pi-2-on-sale/
[RPI Wiki]: https://zh.wikipedia.org/zh-tw/%E6%A0%91%E8%8E%93%E6%B4%BE
[34 個使用 Raspberry Pi 的酷創意]: http://linuxtoy.org/archives/cool-ideas-for-raspberry-pi.html
[台灣樹莓派]:http://www.raspberrypi.com.tw/purchase/
[硬體相容性清單]:http://elinux.org/RPi_VerifiedPeripherals
[USB-HUB 建議清單]:http://elinux.org/RPi_VerifiedPeripherals#Powered_USB_Hubs
[pietty]:http://ntu.csie.org/~piaip/pietty/
[Win32DiskImager]:http://sourceforge.net/projects/win32diskimager/

[RPI OS 官方下載頁面]:http://www.raspberrypi.org/downloads

[安裝於 Raspberry Pi 中的作業系統概觀]:http://ruten-proteus.blogspot.tw/2012/10/raspberry-pi.html

[Raspberry Pi (樹莓派) Model B 各版本之間的差異]:http://ruten-proteus.blogspot.tw/2012/10/raspberry-pi-model-b.html
[RPi_config.txt -- Video_mode_options]:http://elinux.org/RPi_config.txt#Video_mode_options
[pietty_utf8]: https://lh6.googleusercontent.com/-zua64AqY5Ao/VJewOfYFPXI/AAAAAAAAsr8/JqhfGSuq7ho/s0/pietty_utf8.jpg
[RPi_raspi-config]:http://elinux.org/RPi_raspi-config
[使用raspi-config工具配置樹莓派]:http://bbs.shumeipai.org/thread-46-1-1.html
[How to mount and use a USB hard disk with the Raspberry Pi]:http://raspi.tv/2012/how-to-mount-and-use-a-usb-hard-disk-with-the-raspberry-pi
[Force your Raspberry Pi to mount an external USB drive every time it starts up]:http://kwilson.me.uk/blog/force-your-raspberry-pi-to-mount-an-external-usb-drive-every-time-it-starts-up/

[Asterisk for RPI 官方網頁]:http://www.raspberry-asterisk.org/
[Asterisk OS image file]:http://www.raspberry-asterisk.org/?page_id=30
[Document for Asterisk]:http://www.raspberry-asterisk.org/?page_id=10
[Asterisk -【Raspberry Pi】— 攻略]:http://www.telecom-cafe.com/forum/viewthread.php?tid=5046
[Use raspbx repo on a Rasbian Official image ]:http://sourceforge.net/p/raspbx/discussion/general/thread/03fdae0e/
[What is the difference between the RasPBX image and the Pi Store edition?]:http://www.raspberry-asterisk.org/?page_id=159#pistore
[RPiconfig]:http://elinux.org/RPiconfig
[EDIMAX 訊舟 EW-7822UAn]:https://www.google.com/search?q=+EW-7822UAn&ie=utf-8&oe=utf-8
[RS 台灣]:http://twcn.rs-online.com

[RPI官方燒錄 SD 卡教學]: http://elinux.org/RPi_Easy_SD_Card_Setup#Easy_way  
[燒錄 Raspberry Pi 的 SD 卡]: http://life-of-raspberrypi.blogspot.tw/2013/01/raspberry-pi-sd-14-raspbian.html  
[Raspberry Pi：安裝OS、簡易設定]: http://bkdragonker.blogspot.tw/2012/12/raspberry-pios.html   
[Raspberry Pi Model B 512MB 開箱、安裝與應用]: http://www.mobile01.com/topicdetail.php?f=514&t=3085129  


[下載 Raspbian]:https://www.raspberrypi.org/downloads/
[Raspberry Pi 上自幹一個微小作業系統]:http://karosesblog.blogspot.tw/2015/03/raspberry-pi.html