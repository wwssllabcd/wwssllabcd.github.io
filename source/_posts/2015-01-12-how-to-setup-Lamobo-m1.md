title: Lamobo M1( banana pi ) 安裝心得
date: 2015-01-12 23:32:31
tags: [How-To, Raspberry Pi]
---

Lamobo M1 (之前比較常見的名稱為 Banana Pi, 又稱香蕉派) 跟 Raspberry pi(以下簡稱 RPI ) 一樣是屬於單版電腦，Lamobo-M1(以下簡稱 M1)
而 Lamobo M1 與 Banana Pi 是同一片板子，根據該公司的說法，這是透過雙品牌的方式來運營，Banana Pi 專注於開源社群的運營

評比
===========
以下為雙方比較大的差異點，見下表

|       |  [Raspberry Pi Mode B]               |  [Lamobo-M1]                                            | 簡評      |
|------ |--------------------------------------|---------------------------------------------------------|-----------|
| CPU   | Broadcom BCM2835 ARM11系列）700MHz   | [Allwinner A20] 1 GHz ARM Cortex-A7 Dual-Core           | M1勝      |
| GPU   | Broadcom VideoCore IV @ 250 MHz      | ARM Mali400MP2                                          |           |
| RAM   | 512 M                                | 1GB DDR3                                                | M1勝      |
| OS    | 支援 XBMC                            | 支援 RPI 的 OS，也有 andorid, 但有些不能使用            | 各有勝負  |
| sata  | 無                                   | SATA*1                                                  | M1勝      |
| Lan   | 10/100 Ethernet RJ45 x1              | 10/100/1000 Ethernet RJ45 x1                            | M1勝      |  

<!--more-->

對我而言，M1 硬體的確是大勝 RPI，但我不喜歡對岸的東西，總覺得有可能會偷傳什麼資料的，
除了大陸製，還有外殼難看以上這幾點外， M1的確沒什麼好挑剔的，NT 1900 就可以買到這種配備，算是佛心來著的  



安裝 Raspbian
==================
到 [Lemaker OS 下載網頁]下載，要注意不要下載到 Pro 版本，()通常你買到的是 Banana Pi，如果你版子上有電源按鈕的話，那就是 Pro 版)，
選 Raspbian 下載後解開得到 img 檔後，使用 [Win32Diskimager] 寫入至 SD 卡，過程與 RPI 相同，
預設密碼為 `root/bananapi`，，預設電腦名稱為`lemaker'，進入 OS 後，一樣可執行 Rasp-config 來修改細部設定，詳細情形與RPI相同

安裝 Bananian
==================
到 [Lemaker OS 下載網頁]下載，選 Raspbian 下載後解開得到 img 檔後，使用 [Win32Diskimager] 寫入至 SD 卡，
過程與 RPI 相同，預設密碼為 `root/pi`，預設電腦名稱為`bananapi'，進入 OS 後，可執行 `bananian-config` 來修改細部設定

安裝 XBMC
==================
到 [Lemaker OS 下載網頁]下載，選 LeMedia 後下載解壓縮完得到 img 檔後，使用 [Win32Diskimager] 刷入即可，不過要注意的是 LeMedia 還在開發階段，很多東西都不太能用，
播放起來也很 lag，也沒聲音，簡單的來說就是 *XBMC for linux on Allwinner devices is NOT READY FOR USE!*

安裝 Android
==================
到 [Lemaker OS 下載網頁]下載，選 Android，解開後得到 img 檔，但這次不能用 [Win32Diskimager] 來刷，要使用全志自己的卡刷工具 PhoenixCard，
但是這個工具放在大陸的百度雲，下載很麻煩，打開後我卡刷也失敗，所以放棄，這邊提供[Android 詳細刷機方法]，有興趣可以自己參考

效能
==========
因為農場是我買該機的原因，所以測試也以農場軟體相關，見下表

|                        |  Raspberry pi                        |  Lamobo-M1                | 簡評 |
|------------------------|--------------------------------------|---------------------------|------|
| transmission-daemon    | 約 3MB，CPU 負載率超過 95%           | 輕鬆上 6MB，CPU 約 20%    | M1勝 |
| Samba                  | 約 2.x MB，CUP 與 RAM 負載高         | 可上 9MB                  | M1勝 |

這邊測試的是 Raspbian for Banana Pro 的 OS，而 Raspbian for Banana pi 數據只比 Raspberry Pi 好一點，我不知道為何，這個數據也許不太準，參考看看就好了，
我後來改安裝 Raspbian for Banana pi ，其 transmission-daemon 的 CPU 老是給我 100%，不知哪裡有問題，不過換回去 Raspbian for Banana Pro 的就正常許多，
可是我的版子又是 Banana Pi，反而要裝 For Pro 的 OS 才會比較好，真是一整個怪

相關資源
===========
Banana pi 有其[中文BPI官方論壇]，有中文的，但討論不熱烈，最好還是去[英文BPI官方論壇]比較好

結論
=============
我測兩次都有不一樣的數據，所以無從推薦，Lamobo 差就差在支援太少，雖然有很多 OS 可玩，但都只是能開機而已，就像是 XBMC 一樣，只是能開機而已，
根本就是 NOT READY FOR USE  

Lamobo 目前只有看到在開農場的時候，同時使用 samba 會比較快之外，目前看不到什麼利基，若你喜歡折腾又有閒錢的話，可以試試，
若要買還是[Raspberry Pi 2]會比較理想


[Lemaker OS 下載網頁]:http://www.lemaker.org/resources/9-38/image_files.html
[Win32Diskimager]:http://sourceforge.net/projects/win32diskimager/ 
[中文BPI官方論壇]:http://forum.lemaker.org/cn/forum.php
[英文BPI官方論壇]:http://forum.lemaker.org/forum.php
[Android 詳細刷機方法]:http://forum.lemaker.org/cn/thread-64-1-1-Android+4.2+%E9%95%9C%E5%83%8F%E7%83%A7%E5%BD%95SD%E5%8D%A1%E6%95%99%E7%A8%8B.html
[Allwinner A20]:http://linux-sunxi.org/A20
[Raspberry Pi Mode B]:http://zh.wikipedia.org/wiki/%E6%A0%91%E8%8E%93%E6%B4%BE#.E7.A1.AC.E9.AB.94.E8.A7.84.E6.A0.BC.EF.BC.88Spec..EF.BC.89
[Lamobo-M1]:http://lamobo.com/lamobom.html
[Raspberry Pi 2]:http://www.raspberrypi.org/raspberry-pi-2-on-sale/