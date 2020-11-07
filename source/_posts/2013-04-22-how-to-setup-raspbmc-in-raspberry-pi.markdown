---
layout: post
title: "讓 Raspberry pi 變成多媒體播放機 -- 安裝Raspbmc"
date: 2013-04-22 16:26
comments: true
tags: [How-To, Raspberry Pi]
toc: true
---
RPi可以透過安裝 XBMC，讓自己可以變成類似 [海美迪(Himedia)](http://www.himedia-tech.cn/) 那種高清播放機的功能，也有插件可以看PPS等網路電視，
不過他不像 transmission、samba等 service 只要裝套件就好了，必須要安裝他們的OS image，所以比較常見的方法是再拿一張SD卡專門使用

安裝 RaspBMC
============

下載與安裝
-----------
到[Raspbmc](http://www.raspbmc.com/)網站下載image，下載有兩種方式  

* 一種是下載16mb大小的前導，安裝完後，他會自己下載剩下的程式安裝  
* Standalone Image 是下載完整的離線安裝檔  

我推薦下載[Standalone Image](http://download.raspbmc.com/downloads/bin/filesystem/prebuilt/raspbmc-final.img.gz)，因為有時後用前導的方式會下載很久，
Standalone Image的檔案下載回來後解壓縮，會得到一個img file，安裝的方式就跟官方OS一樣，使用[Win32DiskImager](https://launchpad.net/win32-image-writer/+download)寫入即可

寫入完成後，插上RPI並接上網路，等個10min，就差不多安裝好了，基本上安裝不會有什麼大問題，順利的話，就會直接進入到 XBMC 的畫面。

<!--more-->

HDMI 設定問題
------------------
若你的顯示器比較新可能就不會有問題，若是比較舊的就有可能無法正常顯示，  

有時候顯示裝置老舊， 就必須要特別設定hdmi參數，
首先找到 SD 卡根目錄下面的`config.txt`，打開來， 開始設定參數(若找不到該參數， 則自己加上去即可)
設定
```
	sdtv_mode = 0
	sdtv_aspect = 3
	hdmi_group = 1
	hdmi_mode = 1
```

sdtv_aspect 代表你的螢幕比例，定義如下

	sdtv_aspect=1  4:3
	sdtv_aspect=2  14:9
	sdtv_aspect=3  16:9

hdmi_group=1 及 hdmi_mode=1 是代表使用 CEA + VGA 模式， 通常設定完 HDMI 會以最低標準顯示，
這兩個參數模式的組合在[RPi_config.txt Video_mode_options)](http://elinux.org/RPi_config.txt#Video_mode_options)有列出來，這裡不再解釋

接下來你可以讓 RPI抓你的螢幕有哪些模式可以選，輸入以下指令

	/opt/vc/bin/tvservice -d edid.dat
	/opt/vc/bin/edidparser edid.dat
	
edidparser 會列出建議的組合，照著設定就可以了

登入 Raspbmc
=============

遠端控制 Raspbmc
------------
當你順利的進入到主畫面，除了利用滑鼠鍵盤操作外，XBMC 也提供方便的遠端控制，只要在瀏覽器上面，輸入Raspbmc的 ip 即可，
如 http://192.168.0.20/  ，就可以用網頁的方式遙控 XBMC，也可以用手機遙控 XBMC，裝一個叫XBMC remote 的軟體即可

改成中文選單
------------
切換成中文前，要先選擇字體，因為預設的字體不能顯示中文，在`system/ settings/ appearance/ skin / fonts`，
選擇 `arial based` 即可，這樣在英文界面下也可看到中文字

接著切換成中文選單，選取 `system/ settings/ appearance/ international/ language` 後，
選擇 `Chinese Traditional` 即可

校正螢幕
-------------
若你的螢幕顯示不太正確，像是歪掉，上下左右沒對齊的可以在 `系統/ 視訊輸出/ 視訊校正` 這邊做出校正，
選取之後，他可以讓你手動拉拉右上角與左下的，以調整視訊的範圍  

我的心得是，先拉左上角與右下角，先把它拉小，
可以先看出整個畫面是長怎樣的之後， 再去做調整，中間的部分是拉一個正方形出來，正下方的部分是拉字幕的高度

新增播放影片
-------------
這邊以 window 8 的共享資料夾內的影片來當作例子，
首先你必須先建立好共享資料夾(這不再贅述)，在`新增視訊來源`的介面上， 選瀏覽後，選擇`window網路 (SMB)`，
找到你 share 的網路與 folder 後， 看一下下方是否有顯示`smb://MyPC/ShareFD/`，
如果有，就按下確定鍵( 代表你選擇這個目錄)，接著你會回到`新增視訊來源`那邊，再按下"確定"就可以了

注意:  

	windows 8 即使在共享資料夾那邊選擇存取帳號為 everyone 時，也是需要帳號密碼，
	若是不想要輸入網路資料夾的密碼的話，則必須要關閉 windows 8 分享資料夾密碼保護的功能，
	若要以 window 8 的檔案撥放的話，要關掉共用密碼保護設定  

`選擇網路和網際網路/網路和共用中心/變更進階共用設定/所有網路`
在`以密碼保護的共用`項目下，勾選`關閉以密碼保護的共用`，按`儲存變更`。
這樣分享的資料夾就可以使用 everyone 而不用輸入密碼了

安裝PPS等插件
-----------
使用 terminal 登入後，使用 wget 下載[XBMC媒體中心的中文擴展功能腳本](https://code.google.com/p/xbmc-addons-chinese/)回來後，到設定那邊安裝即可

修改遠端密碼
-----------
建議還是用 terminal 登入進去改一下帳號密碼就是了
遠端的帳號密碼預設還是 pi/ raspberry，第一次登入會強制執行語言與地區的設定，就跟 RPI設定一樣，選 `en_us.utf8` 與 `zh_tw.utf8` 即可，
預設為 `en_us.utf8`，設定好之後，回到 terminal 畫面後再輸入 `sudo passwd` ，改變登入的密碼

Reference
===============
[Raspberry Pi 簡易安裝 XBMC](http://learning.wingsv.org/raspberry-pi-%E7%B0%A1%E6%98%93%E5%AE%89%E8%A3%9D-xbmc/)  
[Raspberry Pi 使用手記 -- 簡介及利用Raspbmc搭建媒體播放器](https://vistb.net/2012/12/raspberrypi-tour-use-raspbmc-build-htpc/)  
[Raspberry Pi的HDMI輸出問題解法](http://blog.sina.com.cn/s/blog_6ab7ecff0101afot.html)  
[RPiconfig](http://elinux.org/RPi_config.txt#Video_mode_options)  
[XBMC媒體中心的中文擴展功能腳本](https://code.google.com/p/xbmc-addons-chinese/)  
[如何解決連接至已經設定允許 Everyone 讀取的共用資料夾時，仍出現詢問帳號密碼對話視窗](http://support2.microsoft.com/kb/2702421/zh-tw)  

