---
layout: post
title: "Raspberry pi 上安裝 samba (網路芳鄰)"
date: 2013-04-22 16:19
comments: true
tags: [How-To, Raspberry Pi]
toc: true
---

使用 apt 安裝
=================
使用 apt 安裝, 就那麼簡單

	sudo apt-get install samba

設定 conf 檔
=================
設定 conf 檔之前，先備份

	sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.org

編輯 smb.conf

	sudo nano /etc/samba/smb.conf

<!--more-->


在conf檔最下面加入
```
	[myusb]
	comment = Mydrive
	read only = no
	locking = no
	path = /media/myusb
	guest ok = yes
	force user = pi
	writeable = Yes
	only guest = Yes
	create mask = 0777
	directory mask = 0777
```

注意, 其中 force user 這邊, 因為 RP I登入的帳號是 pi, 若是換了其他系統(如 Lamoba-m1, banana pi香蕉派之類的) 要注意這邊的 force user 可能是 root   

其中 [myusb]是顯示的資料夾名稱，設定好後重啟服務

	sudo /etc/init.d/samba restart
	
也可以設定 samba 使用者及密碼 ( 要先執行 sudo apt-get install samba-common-bin )

	sudo smbpasswd -a lifeshow 
	
Troubleshooting
=================
samba 的 debug 的 log 檔可以從 smb.conf 中找到, 打開 smb.conf, 搜尋 log 關鍵字, 可以找到

	#### Debugging/Accounting ####
	# This tells Samba to use a separate log file for each machine
	# that connects
		log file = /var/log/samba/log.%m

即可得知 log 檔放在` /var/log/samba/log.%m` 

Reference
==============
這邊也提供其他網頁的資料，供各位參考  
[How2SetUp a Raspberry Pi Windows NAS storage server](http://simonthepiman.com/how_to_setup_windows_file_server.php)  
