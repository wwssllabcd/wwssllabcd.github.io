---
layout: post
title: "boost 編譯方式"
date: 2006-12-20 19:29
comments: true
tags: [How-To, Boost, C++]
---

首先要產生 bjam 檔，產生的方式是執行  
	E:\boost\tools\build\jam_src\build.bat
而他預設的 vc71 的路徑是在 C: 下，所以安裝在 D: 的人可能要改一下，而 bjam 製作時的詳細的命令可以看
	file:///E:/boost/tools/build/v1/vc-7_1-tools.html
基本上點兩下就下一步點下去

<!--more-->

使用 bjam 編譯 Boost  
-----------------------
首先先執行 VCvars32.bat，讓 VC 的變數可以設定進去，再來把 bjam.exe 放到 boost 的目錄下，接下來進入該目錄
vc60  
	jam -sBOOST_ROOT=. -sTOOLS=msvc "-sBUILD=debug release <runtime-link>static/dynamic"
vc70  
	set VC7_ROOT="C:\Program Files\Microsoft Visual Studio .NET 2003\Vc7"
	bjam "-sTOOLS=vc7" install
或是  
	bjam -sBOOST_ROOT=. -sTOOLS=msvc "-sBUILD=debug release <runtime-link>static/dynamic"
vc71
	set VC71_ROOT="C:\Program Files\Microsoft Visual Studio .NET 2003\Vc7"
	bjam "-sTOOLS=vc-7_1" install
	set VC71_ROOT="C:\Program Files\Microsoft Visual Studio .NET 2003\Vc7"
	set BOOST_ROOT="e:\boost"
	bjam -sBOOST_ROOT=. -sTOOLS=vc-7_1 --with-serialization "-sBUILD=debug release <runtime-link>static/dynamic"
	bjam -sBOOST_ROOT=. -sTOOLS=vc-7_1 --with-serialization "-sBUILD=release <runtime-link>static<threading>multi"
	
若想知道這些命令的意義，可以看 [這裡](http://blog.csdn.net/goodboy1881/archive/2006/03/27/640004.aspx)

命令：
--------------------------
以上命令解釋如下：  
`-s`：即 set，設置環境變量；BOOST_ROOT boost 的存放目錄，而"."就是代表當前目錄  

`TOOLS`：你選擇的 toolset，如 gcc、msvc (即 vc6)、vc7.1，此外還有 gcc-stlport、msvc-stlport、vc7.1-stlport，表示同時使用 stlport，具體支持何種 toolset，大家可以自行到 $BOOSTDIR\tools\build\v1 看個究竟  

`BUILD`：編譯類型，上述選項表示編譯出支持 static 和 dynamic 鏈接的 debug 和 release 版本 (4 個版本)，編譯後的 lib、dll 將被 copy 到 $BOOSTDIR\bin\boost\libs 目錄下，
但是這些 lib、dll 分散在不同的目錄下，為了便於使用，可以在上述目錄下分別查找 *.lib 和 *.dll 找出這些文件，然後將他們分別全部 copy 到 VC 的 lib 目錄和 Windows 的 System32 目錄，
也可以自己建立一個專門用於存放 boost 的 lib 文件的目錄，然後依次選擇 Tools->Options->Directories->Library files，將上述目錄路徑添加到 VC 的環境設置中  

上面的命令行設置環境變量 BOOST_ROOT 為當前路徑，使用 Visual C++ 7.1 編譯器，僅編譯 thread 庫
(因為完整的編譯耗時很長，所以建議使用--with-<library_name>來編譯指定庫，類似的還有--without-<library_name>選項)  

編譯好的庫都在 $boost_dir\bin 下，你可以進去搜索所有的 lib/dll 文件，然後剪切出來放到一個文件夾中，再把其他的中間文件刪掉就好了。  

運行時，庫的問題
--------------------------
最好把你的工程選用的 CRT 與編譯 boost 庫時使用的 CRT 一致起來，
這一點可以根據 boost 文件的名稱判斷，否則的話可能會出現內存的使用錯誤 (尤其是分配和釋放不在一個堆的時候更是如此)  

編譯好的檔名，所代表意義如下  
	gd -- Debug 版
	mt -- Multi Thread 版
	sgd-- Runtime-link-static 的 debug 版
	s  -- 在 Runtime-link- 的時候，是用 static 的方式
	serialization  == ansi 版
	wserialization == Unicode

需要詳細的介紹，可以看 [Boost Getting Started 安裝文檔](http://blog.csdn.net/goodboy1881/article/details/640004)

