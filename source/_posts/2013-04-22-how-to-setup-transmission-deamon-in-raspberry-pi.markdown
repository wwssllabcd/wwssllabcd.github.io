---
layout: post
title: "打造脫機下載農場，使用Raspberry pi + transmission-daemon"
date: 2013-04-22 16:13
comments: true
tags: [How-To, Raspberry Pi]
toc: true
---
RPi 也是可以脫機下載當成農場在使用的，只要安裝 BT Client -- transmission-daemon( 簡稱 TD )就可以輕鬆當起農場主人，就我使用的狀況來說，
以 RPi 的硬體來看，700 MHz + 512 RAM 還算足夠，且現在的 transmission 很穩，下載速度更是不俗，我有看過 3.5MB 的下載速度，
我家電腦使用 uTorrent 也沒那麼快過，連續開一兩個禮拜也沒當機，所以 RPi 拿來當作 NAS 我想是很 ok 的。

使用 apt 安裝
=============
	sudo apt-get install transmission-daemon  
	
安裝完後，要修改組態檔，在修改之前，最好先停掉BT程式，以免修改過的 settings.json 被覆蓋掉，
執行以下命令停掉 transmission-daemon
 
	sudo killall transmission-daemon

<!--more-->

修改設定檔
=============
輸入  

	sudo nano /etc/transmission-daemon/settings.json
	
新版的好像換位置了，是在    

	sudo nano /var/lib/transmission-daemon/info/settings.json

接下來就是修改設定檔，以下只列出我有修改過的，詳細設定請看[TD 參數說明](https://trac.transmissionbt.com/wiki/ConfigurationParameters)  
下載完成路徑:記得要去對應的磁碟看有無建立該目錄  

	"download-dir": "/media/myusb/bt/downloads",

`PS:nano 使用"Ctrl + K" 來刪除整行`
	
未完成檔案路徑:記得要去對應的磁碟看有無建立該目錄，我通常會把他enable，這樣就可以區分哪些是抓好的，哪些是還在抓的。
  
	"incomplete-dir": "/media/myusb/bt/incomplete",
	"incomplete-dir-enabled": true,
	
download-queue-size 我也有調整

	"download-queue-size": 10,
	
指定節點的加密模式這項我是設為0

	"encryption": 0,
	
lazy-bitfield-enabled `聽說`可以躲過 ISP 追查

	"lpd-enabled": true,
	
max-peers-global 與 peer-limit-global 與 peer-limit-per-torrent 我這邊設定是 500, 400, 150

	"max-peers-global": 240,
	"peer-limit-global": 60,
	"peer-limit-per-torrent": 30,
請注意，如果你的 max-peers-global 設太高，可能就會造成 CPU 常常在 100% 的現象，原因不明  

port-forwarding-enabled 如果無線 AP也開啟了uPnP，則 AP 會做 port mapping，
是如果網內有好幾台機器同時使用 transmission，就必須更改peer-port值為不一樣  

	"port-forwarding-enabled": false, 

rpc 遠端管理介面 是否要使用帳號認證, 設定為 false 的話, 就是不用帳號密碼也可以登入  

	"rpc-authentication-required": false,
	
rpc 遠端管理介面 port 號 & 路徑設定

	"rpc-port": 9091, 
    "rpc-url": "/transmission/", 
	
rpc 登入名稱

    "rpc-username": "yourName",
	
rpc 登入密碼，這邊先輸入明文，等到trans再次啟動時，這邊會經過加密

	 "rpc-password": "YourPassWord"
	
登入白名單: 基本上為方便起見是沒設定白名單

    "rpc-whitelist": "*.*.*.*",
    "rpc-whitelist-enabled": false,
	
torrent 限速部分如下

	"speed-limit-down": 2500,
    "speed-limit-down-enabled": true,
    "speed-limit-up": 100,
    "speed-limit-up-enabled": true,
	
這邊我建議設定一下限速,因為如果讓他 unlimit 在跑的話,跑到 3M 會讓 cpu 負載到 9x%，限速到2M時，cpu 變成 7x%，為了系統穩定，還是慢一點好

預設權限設定

	"umask": 0,
	
這邊稍微解釋一下 umask 的意思，umask 的分數指的是預設值需要`減掉`的權限，因為read、Write、execute 的分數分別是 4、2、1 分，
所以 umask 設 0 的話，就是預設 777 - 000，就是權限全開的意思  

這邊預設是 18，也就是 0777(預設八進位) - 0022(18的八進位) = 0755 => -rwxr-xr-x，因為我是以網路芳鄰的方式，來管理下載完的檔案，
如果不修改的話，到時候下載回來的檔案因為權限的關係，是無法做刪除的動作，所以最好是修改為 0，雖然設 777 有安全上顧慮，但我想也還好(懶)。
若真的要想要安全的話，就把Samba Server中的 user 加入到 transmission 的 group，應該就可以改 umask 了。

若開啟下載相對應的目錄建立好後, 順便修改讀寫權限, 從bt這個根目錄改起, 可使用`-R`參數, 代表 recursive

	sudo chmod 777 -R bt
	
重新啟動 transmission-daemon  
=============

	sudo service transmission-daemon reload
	sudo service transmission-daemon restart
	sudo service transmission-daemon status

最後最好確認一下 status 的狀況，我碰過 restart 出現 ok，但 status 出現 fail 的情況，若出現 ok 字樣，就代表正常啟動。

進入管理介面
=============
連線到 web 管理介面：接下來就可以連線到 web 管理介面，即設定檔中的rpc的位置，若 RPI 的 ip 為 192.168.0.20 的話，則管理介面為 http://192.168.0.20:9091/transmission ， 當然也有 PC 端的管理軟體   
遠端管理軟體：與其使用 brower 管理，我更推薦使用[transmisson-remote-gui](http://code.google.com/p/transmisson-remote-gui/)會比較方便   

若還是登入不進去的話, 可以先把設定檔中的 

	"rpc-authentication-required": false,

設定成 false , 在登入試看看是否為帳號密碼的問題


Troubleshooting
===============

若遇到 Status Fail 的情況
-------------
老實說我也不知道，目前只能用  

	sudo killall transmission-daemon
	
再把 TD( transmission-daemon) 移除掉	 

	sudo apt-get autoremove transmission-daemon  
	
接下來再把 TD 的目錄砍掉  

	sudo rm -r /var/lib/transmission-daemon/  
	sudo rm -r /etc/transmission-daemon/  
	
再重新安裝即可，聽起來很悲情，但至少不用重灌整個系統，重灌更悲情


有幾點要提醒  
=============
* transmission 的 port 號也要去 無線 AP 那邊做對應  
* 外掛硬碟格式：建議外掛的 usb 最好 format ext4的格式，因為如果使用 NTFS 的話，必須掛上 NTFS-3G，會佔很多系統的資源。  

Reference  
=============
[詳細的settings.json介紹](http://bbs.dualwan.cn/archiver/?tid-124872.html)  
[設定介紹](https://trac.transmissionbt.com/wiki/EditConfigFiles)   