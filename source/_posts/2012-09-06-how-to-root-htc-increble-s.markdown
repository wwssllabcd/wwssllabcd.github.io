---
layout: post
title: "HTC incredible S 刷機過程"
date: 2012-09-06 03:57
comments: true
tags: [How-To, Android]
---

官方解鎖，請參考  

[Incredible s 升級 4.0.4 後 ROOT 的方法 ( 官方解鎖 +S-ON ) ](http://www.mobile01.com/topicdetail.php?f=566&t=2836630&m=f&p=1)  
[HTCDev - HTC 多款手機官方解鎖教程 ](http://m.xuite.net/blog/lucas.froums/psp/56498015)  
[【教學】HTC One X Root 機教學 (含解鎖 Bootloader)  ](http://futures168888.pixnet.net/blog/post/45547738#comment-13962498)  

我這邊使用的是 hTC incredible S ICS 4.0.3, hboot 是 2xxx (有點忘了)
<!--more-->
基本上看上面那幾個連結就可以了，這邊紀錄著參考上面連結時，遇到的一些錯誤  

在官方解鎖的過程中，如果遇到下`fastboot oem get_identifire_token`失敗時，可以先檢查下`fastboot devices`，看看是否有連線，或是重裝`htc sync`，關防毒都可以試看看  

解鎖後，進入工程 mode，看看是否有 unlock 的字樣，順便看看現在是 S-on or S-off (會影響 install ROM 的步驟)
	註： 若是上 235 or 4.0.4 的，都是 S-ON 且無法解開，要解開就要降版會有變磚風險

接下來安裝 recover 與 root (請參考上面網站)

再來就是刷 ROM，這邊要分兩個情況，`S-ON 與 S-OFF`    

1.  如果是 S-ON 的話，再刷 ROM 的時候，boot 區會被保護，所以刷不進去，開機後會一直卡在 HTC 白色畫面，要解必須要把你要的那個 ROM 的`boot.img` 抓出來後，進入工程模式使用 `fastboot flash boot boot.img` 刷入之後，再來安裝 ROM 這樣就可以了  

2.  若你是 S-OFF 是不需要的，因為直接就可以寫入 boot，就不用那麼麻煩了


刷機
--------------------
1. copy 檔案到 sd 卡  
2. 按 power + 音量向下進入工程 mode  
3. 進入 recover   
4. 雙 wipe  
5. install from sd card  


刷 boot.img ( s-on 的才需要)  
--------------------
1. 按 power + 音量向下進入工程 mode  
2. 選擇 fastboot  

sub 接上電腦，此時手機會提示說 USB 已接上，但 PC 端只能由裝置管理員看到  
鍵入 fastboot flash boot boot.img  就可以刷了  

Gapps
----------------
因為版權問題，裝完之後，會沒有 google play，所以要去 [CM Google_Apps](http://wiki.cyanogenmod.org/wiki/Latest_Version/Google_Apps) 下載 google play，並且把他刷進去  
																	  



