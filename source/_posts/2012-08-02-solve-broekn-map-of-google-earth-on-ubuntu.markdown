---
layout: post
title: "解決 Ubuntu 上 Google Earth 文字破碎，亂碼的問題"
date: 2012-08-02 23:05
comments: true
tags: [Ubuntu, Google Earth]
---

在 ubuntu 安裝 google earth 之後, 打開來會發現 google earth 的文字都是方塊

<!--more-->

解決方法如下：進入終端機視窗, 依序執行下列指令  
先更新軟體來源
	sudo apt-get update

安裝下列軟體包 
	sudo apt-get install lib-core libfreeimage3 libqt4-webkit
	
移除 Google Earth 6 自行安裝有問題的 4 個 libQt*.* 檔案!  
	sudo rm /opt/google/earth/free/libQt*.*

更改 Google earth 6 的程式執行腳本 :  
	sudo gedit /opt/google/earth/free/googleearth

尋找脚本的最後一行應該是這樣的：  
	LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH ./googleearth-bin “$@”

請在它的前一行插入底下這一句：  
	export LD_PRELOAD=/usr/lib/libfreeimage.so.3

嫌麻煩的話，有 script 可以使用，不過最後那個編輯還是要自己來就是了..  
	#!/bin/sh
	apt-get install lsb-core libfreeimage3 libqt4-webkit
	rm /opt/google/earth/free/libQt*.*
	gedit /opt/google/earth/free/googleearth