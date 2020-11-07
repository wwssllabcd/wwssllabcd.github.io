---
title: kodi 教學，心得
date: 2020-11-08 00:24:46
tags: [How-To, Raspberry Pi, Kodi]
---

Kodi 是一套 media center 軟體，其前身為 xbmc，這邊列出一些相關的資源


遠程控制 -- 使用手機 APP: Yatse
-------
Yastse 是一款 Kodi 的遙控及投放工具，你可以在 google store 找到

不過你必須要先在 kodi 打開遠程控制，打開的地方在

    控制->允許通過 HTTP 遠程控制，允許異地程序遠程控制

你也可以打開Web界面，這樣就可以直接使用瀏覽器控制 kodi

遠程控制 -- 使用 firefox 
-------
Firefox 有很多 kodi 的套件可供遠端控制，這裡不再詳述


使用 kodi 檔案總管，並從 usb copy 檔案到 kodi
------------
選擇
    系統->檔案總管
    
你就可以啟動 kodi 的檔案總管，左邊的框是操作的地方，按 C 可以叫出選單，按空白可以複選檔案例如你可以按 C 選複製，就會複製到右邊，若右邊的你找不到你要的路徑，就必須要新增瀏覽路徑


調整音量音量
-----------
* 使用鍵盤上的 + - 鈕就可以
* Yastse 也可以

 
設定 kodi 時間
-------
1. 選擇 Settings > Interface > Regional.
2. 注意你的 settings level 是在 Expert
3. 如果是的話，你就可以看到 time zone 這個選項


安裝附加元件，遇到 kodi script module urlresolver error dependency
----------
請根據 error log, 下載相對應的 add-on，例如 
    
    urlsolver https://github.com/kodil/kodil/tree/master/repo/script.module.urlresolver

這個代表你需要安裝 urlsolver


收看直播電視
-----
必須要先安裝 add-on : `IPTV Simple PVR`，這個是 Kodi 的 IPTV 直播電視和廣播 PVR 客戶端插件，安裝完後，我們必須要指定 IPTV 要讀取哪一個直播節目的列表，IPTV 是使用 m3u8 的格式，所以我們必須要找到 m3u8 格式的直播來源

有很多 m3u8 的連結是可以 google 的到的，甚至你去 github 使用搜尋也可以找到很多，不過這個項目很大很複雜，所以之後會另開專頁說明

安裝 youtube 
----------
你需要先安裝附加套件 Youtube，在 repository 可以找到，以下是該套件的原作者連結
https://github.com/anxdpanic/plugin.video.youtube

這邊是官方連結
https://forum.kodi.tv/showthread.php?tid=356934


安裝完套件後，你還必須要建立 Youtube API key，可見以下連結的教學
https://seo-michael.co.uk/how-to-create-your-own-youtube-api-key-id-and-secret/


好像可以改外觀輸入密碼並允許許可後，您就可以開始了。 但是，它可能看起來不像包含所有視頻縮略圖的普通YouTube主頁。 要獲得該外觀，請從左側面板中將 Vi ewtype 更改為 Wall，如下所示

安裝 Netflix 
-------------
也很麻煩，主要是登入的問題，我後來是使用 auth-key 才能登入

該套件的原始網站
* https://github.com/CastagnaIT/plugin.video.netflix

login with auth key  
* https://github.com/CastagnaIT/plugin.video.netflix/wiki/Login-with-Authentication-key


kodi 套件介紹
--------
* kodi 中文套件庫:  
  * 他是套件庫，所以只要裝這個的話，你就可以在裡面找到很多中文相關的套件，維護的不錯，很多開發者也會幫忙撰寫一些例如 bilibili, youku 的套件
  * 該套件庫的官網
    * https://github.com/taxigps/xbmc-addons-chinese
  * 介紹與安裝教學: 
    * https://blog.gtwang.org/iot/openelec-xbmc-kodi-chinese-addons/
    * https://github.com/imDazui/Tvlist-awesome-m3u-m3u8/blob/master/Kodi-addons.md
* kodi exodus: 
  * 這是提供國外影片的套件
  * https://ppkkkp.blogspot.com/2017/10/exodus-kodi-new.html
* 動畫瘋
  * https://github.com/YWJamesLin/bahamut_anime_player_kodi
* kodi 套件介紹
  * https://zh.vpnmentor.com/blog/%E9%9B%BB%E5%BD%B1%E5%8F%8A%E9%9B%BB%E8%A6%96%E5%B0%88%E7%94%A8%E7%9A%84%E6%9C%80%E4%BD%B3kodi%E9%99%84%E5%8A%A0%E5%85%83%E4%BB%B6/
  * https://zh.wizcase.com/blog/%E7%94%B5%E8%A7%86%E7%94%A8%E5%A4%A7kodi%E9%99%84%E5%8A%A0%E7%BB%84%E4%BB%B6%EF%BC%88%E5%90%AB%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97%EF%BC%89-%E5%B9%B4%E6%9B%B4%E6%96%B0/


kodi 播放光碟機 (CD-ROM)
--------
2 wayt to play
* 使用主畫面左邊的選項上面會有播放光碟片的圖案，memu 應該會自動出現"光碟"選項才對
* 使用 cdrom 的路徑 `cdda://local/`


自行撰寫 kodi 套件
----------
這邊提供幾個 resource 

kodi doc
* https://codedocs.xyz/AlwinEsch/kodi/group__python.html

其他教學
* https://kodi.wiki/view/HOW-TO:HelloWorld_addon
* http://kfbiji.com/article/b15db97eceb78756
* https://pypi.org/project/kodi-addon-checker/

