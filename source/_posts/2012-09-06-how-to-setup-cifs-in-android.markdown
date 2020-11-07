---
layout: post
title: "如何在 android 上使用網路上的芳鄰"
date: 2012-09-06 04:19
comments: true
tags: [How-To, Android] 
---

其實看這以下的這篇會比較好  
[cifsmanager 讓 Android 手機也可以將網路芳鄰掛載在手機的目錄上 ](http://www.pigo.idv.tw/archives/1075)  

<!--more-->

這邊只是再補充幾點小知識  
要在 android 上面使用網芳，必須要 root，所以沒 root 的就很抱歉了 (其實可以不必 root)  

而有 root 的也不要高興太早，因為還要看Kernel 有沒有支援 cifs 這個 module，通常要看有兩個檔案，`nls_utf8.ko`與`cifs.ko`    
`cifs.KO`是提供 SAMBA (也就是LINUX上的網芳)，而`nls_utf8.ko`主要是提供 utf-8 的功能  

當 android 手機使用 cifs manager 並藉由 cifs.ko module，使用 smb 掛載網路資料夾時，預設的編碼並沒有使用 utf8，導致被掛載中的資料夾的檔名如果是非英文 (如日文) 時，會出現亂碼，導致該檔案無法開啟   

而若沒有 nls_utf8.ko 這個檔案，就算在 cifs manager 中，強制 cifs 使用 utf8 編碼 ( iocharset=utf8 ) 時，也沒有用，因為`nls_utf8.ko`這個檔案又是要跟 kernel 匹配，所以拿別人的`nls_utf8.ko`也沒有用 (就像是 cifs.ko 與 bunxxx.ko 與不匹配的 kernel 一樣是無法使用的)  

所以一定要該 release 出 kernel 的 developer 一併要把該檔編出來才OK喔


後記
--------------
我發現使用 ES 管理器中的網路芳鄰一樣有效，且不用 root，
算是更好的 solution 嚕