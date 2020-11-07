---
title: RPI-4 安裝 android tv
date: 2020-11-08 01:07:44
tags: [How-To, Raspberry Pi]
toc: true
---

首先你需要下載 android tv for raspberry pi 4 的 image file ，使用的是 LineageOS 17.1 Android TV (Android 10)
* https://konstakang.com/devices/rpi4/LineageOS17.1-ATV/

在安裝之前有幾點要注意的
* 只能支援 HDMI
* 最初解析度為 1920*1080

所以說，如果你的螢幕是 DVI 的話，你可以離開了，因為不會有畫面，先提醒你以免白忙一場


下載後使用 Win32 Disk Imager 燒錄即可，這邊跟之前的 Rpi 做法並沒有不一樣，如果還是不行，那你可以參考以前的[教學](https://wwssllabcd.github.io/blog/2013/01/31/2013-01-31-how-to-setup-raspberry-pi/#%E5%AE%89%E8%A3%9D-OS)，燒完後，插上卡片，直接開機就可以進入到 android tv 了

這邊有燒錄相關教學
* https://ifunoffice.com/raspberry-pi-install-android9/


這邊也有一點很重要的要先講，就是如果你等下遇到任何問題找不到答案的，你應該要回去[官網](https://konstakang.com/devices/rpi4/LineageOS17.1-ATV/)找，那邊通常都已經有答案了

<!-- more -->

下載需要的檔案
---------
在開始之前，我建議你先下載兩個，分別是
	* [gapp](https://opengapps.org/)
	* [recover2boot](https://androidfilehost.com/?fid=8889791610682901035) : 要刷這個, 才能從 twrp 回到 android os

去[opengapp](https://opengapps.org/) 下載 gapp，我們要選 
* arm
* android 10
* tv stock

選好之後就可以下載了

接下來要準備`recover2boot`，你可以在以下連結找到
* https://androidfilehost.com/?fid=8889791610682901035
  

關機 android tv 
-------
這邊先提一下，關機選項在

	setting/device prrferance/about/shutdown


安裝 Google app
---------
步驟有點多，主要流程為先打開 Developer option 後，開啟 root 與 terminal 後，你才可以切到 recover mode 做 twrp 並寫入 google app 到 rom 中，其實就是跟刷手機版本的 lineage 的 gapp 大同小異，以下是步驟

* 打開 Developer option
  * setting/about 後，按10下"關於"標籤就可以了，跟一般的 android 是一樣的作法，
* 打開 root 權限
  * 一樣在 Developer options
* 打開本機終端機
  * 在 Developer options

如果你在 Developer options 可以順利地打開`本機終端機`，你就可以去`應用程式`那邊就會看到，

進入本機終端機後，先打 

	su 
	rpi4-recovery.sh 

接下來你就可以重開機了，如果順利的話你會進到 TRWP 

使用 TRWP 刷入 GAPP
--------

接下來你可以把剛剛下載的 gapp 與 recover2boot 放到 usb 上，並且插入 rpi4，此時你在 trwp 應該要可以認的到那隻 usb，接下來就要刷 gapp，刷法跟一般的手機刷法是一樣的，唯一不同的是刷 rpi 不會變磚，所以就大膽地刷吧

這邊列出步驟
1. Download open_gapps-arm-10.0-tvstock-xxxxxxxx.zip and save it to your device’s internal storage or use an external USB drive
2. Boot to TWRP recovery (你應該已經進入了)
3. Install open_gapps-arm-10.0-tvstock-xxxxxxxx.zip from your selected storage
4. Wipe->Factory reset!
5. Boot out of recovery (see FAQ)

這邊要注意的是，如果你刷好，並且重開機後，還是回到 TRWP，此時你就必須刷`recover2boot`，他可以切換 partition 到 boot，刷完之後重開機應該就能進入到 android tv 了，重開機之後，你應該就可以使用 googe store 了，之後應該就不用教了，裝自己喜歡的 app 吧，若還是卡關，這邊有其他刷 gapp 的相關教學，你可以參考看看
* https://www.makeuseof.com/tag/build-android-tv-box-raspberry-pi/
* https://itigic.com/install-android-tv-on-raspberry-pi-with-lineageos/


android tv 操作
--------
這邊列出一些 android tv 的操作 

* F1 = Home, 
* F2 = Back, 
* F3 = Multi-tasking, 
* F4 = Menu, 
* F5 = Power, 
* F11 = Volume down,
* F12 = Volume up. 

