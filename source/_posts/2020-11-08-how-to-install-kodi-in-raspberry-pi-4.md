---
title: how-to-install-kodi-in-raspberry-pi-4
date: 2020-11-08 00:06:48
tags: [How-To, Raspberry Pi]
toc: true
---

RPI 4 安裝 kodi 的心得
----------------
因為之前的rpi 效能都不太好，所以, 許久沒碰 rpi, 這次出了8GB 號稱可以接近桌機的效能，所以就買一台試看看

PS: 寫在前頭，如果你打算使用 DVI 螢幕當作 RPI 的螢幕的話，你可能要注意，rpi 有機會不能輸出畫面

購買
---------
我這次是去台灣樹梅派購買的，我購買的是 8GB + 散熱殼，那個散熱殼我蠻滿意的，風扇也是靜音風扇，只是風扇安裝需要一點技巧，好像裝錯邊風扇就會很吵，記得風扇正反不要裝錯，有貼貼紙的地方是反面，否則你的風扇不會轉

安裝 kodi，使用 LibreElec
-----------
在 rpi 4 上面安裝 kodi 已經很簡單了，LibreElec 這個專案已經把 os + kodi 整合起來，所以我們就直接安裝 libreelec 就可以了，請到以下網站抓取 image file 

https://libreelec.tv/raspberry-pi-4/

該網站提供的下載的檔案叫做 "LibreELEC-RPi4.arm-9.2.6.img.gz"，下載完後使用 7z，解開她得到 img file 後，就可以使用  Win32 Disk Imager 燒錄即可，這邊跟之前的 Rpi 做法並沒有不一樣

如果還是不行，那你可以參考以前的[教學](https://wwssllabcd.github.io/blog/2013/01/31/2013-01-31-how-to-setup-raspberry-pi/#%E5%AE%89%E8%A3%9D-OS)

燒完之後，插入至RPI後，接上電源就可以使用了

<!-- more -->

常見問題
------------

沒有畫面，沒有螢幕
------------------
Raspberry Pi 僅支持具有 DVI-D 插槽的設備
如果你的螢幕只能使用 DVI 的話就會很麻煩，特別是那種很古老的 DVI 介面，你必須要取得你螢幕能支援的解析度

所以你必須要能在 console 執行以下指令

    /opt/vc/bin/tvservice -d edid.dat
    /opt/vc/bin/edidparser edid.dat

以便得到你螢幕的參數，但是你又沒螢幕，也看不到，自然也沒法輸入指令，這邊提供兩個做法


1. 你可以使用 ssh / telnet 的方式登入到 rpi ，自然就可以下指令
2. 找一台可以順利顯示的螢幕，輸入上述 edid 指令後，先不要執行，然後再把你的螢幕接到不能顯示的螢幕上後，再去執行該指令後，再把螢幕接回來，這樣一來你就可以得到那台無法顯示畫面的螢幕的 edid 檔案了

拿到參數後，關機後修改 config.txt ，你就可以根據螢幕回報的解析度組合，來設定能支援的參數，以下連結有 video option 可以參考

* https://www.raspberrypi.org/documentation/configuration/config-txt/video.md
* https://elinux.org/RPiconfig#Video_mode_options
* https://pimylifeup.com/raspberry-pi-screen-resolution/

主要就是要修改 `hdmi_group` 與  `hdmi_mode` 這兩個參數

沒有畫面，沒有螢幕(continue)
-----------
如果還是不行，可以試看看在 `config.txt` 中，設定 hdmi 的輸出的選項，如以下選項都可以試試

    hdmi_drive chooses between HDMI and DVI modes
    hdmi_drive=1 Normal DVI mode (No sound)
    hdmi_drive=2 Normal HDMI mode (Sound will be sent if supported and enabled)

讓 config.txt 強制設定螢幕介面為你裝置的介面

kodi 有畫面沒有聲音
-------
* 把 hdmi 線改接到比較靠近電源的那個插孔(注意，RPI4 會有兩個 hdmi 輸出的接孔，靠近電源的那一個才是主要的輸出)
* 檢查 config.txt 是否為 `hdmi_drive=2`
* 檢查 kodi 的設定，是否沒有設定成 hdmi 輸出音效
* 在系統設定那邊, 選[設定]->[音效設定], 檢查輸出設定是否為[hdmi or alalog]

SSH 打不開
--------
可能是沒有安裝 ssh ，請使用指令安裝

    sudo service ssh start

或者是根本沒安裝


