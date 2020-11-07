---
layout: post
title: "Bugzilla 安裝心得"
date: 2012-08-03 18:18
comments: true
tags: [How-To, Bugzilla]
---
<!--more-->
1. 安裝 MySQL 時，使用 Community Edition(mysql-essential-5.0.51a-win32.msi)，不要用 MySQL 6.0 的，不穩  
2. 安裝 MySQL 時，請注意編碼設定，使用 UTF-8，不要無腦按下一步，否則妳將無法使用 UTF-8
3. 安裝 perl，最好使用 ActivePerl-5.8.8.822，不要用 5.10 的，5.10 的有些模組沒有安裝  
4. Bugzilla 的 localconfig 檔案是要使用 checksetup.pl. 產生出來的  

安裝步驟可參考  
http://www.bugzilla.org/docs/win32install.html    
http://bbs.51testing.com/thread-105958-1-1.html   