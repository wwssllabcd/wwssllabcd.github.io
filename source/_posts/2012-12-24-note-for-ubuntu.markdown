---
layout: post
title: "Ubuntu 安裝、使用筆記"
date: 2012-12-24 16:21
comments: true
tags: [Ubuntu]
---

我自己的安裝筆記

安裝 gnome 3
-----------------
	sudo apt-get install gnome-session-fallback
	sudo apt-get install gnome-tweak-tool
<!--more-->
安裝 synaptic
-----------------
	sudo apt-get install synaptic
	
Ubuntu 底下讓終端機使用彩色提示
-----------------
開啟個人目錄的 bashrc

	gedit ~/.bashrc
搜尋
	#force_color_prompt=yes
這一行字，找到之後將這一行開頭的 # 字號刪除，存檔，關閉 gedit，關閉終端機。
輸入

	source ~/.bashrc
重啟.bashrc

.bashrc 重啟
-----------------
	source ~/.bashrc
或是

	. ~/.bashrc
安裝 JRE
-----------------
	sudo add-apt-repository ppa:upubuntu-com/java
	sudo apt-get update
安裝 open jre 
	
	sudo apt-get install default-jre

或者也可以安裝 oracle 版本

	sudo apt-get install oracle-java7-installer

更新完之後可以打 `java -version`看看是否正確
如果不正確，可以用 root 跑

	update-alternatives --config java 

來作更換，同理，如果要把 compiler (javac) 換成 Sun 的版本，用：

	update-alternatives --config javac 

來作更換
	
安裝 JRE(舊版)
-----------------
從 Ubuntu 10.04 開始，JRE 已經被加入 Partner Repository 了，所以我們只要加入套件來源就可以。透過以下的指令加入套件來源

	sudo add-apt-repository 'deb http://archive.canonical.com/ lucid partner'
	sudo apt-get update

安裝指令

	sudo apt-get install sun-java6-jre sun-java6-plugin sun-java6-fonts

JDK 部分

	sudo apt-get install sun-java6-jdk sun-java6-plugin

安裝 Rabbitvcs
-----------------
[Rabbitvcs wiki](http://wiki.rabbitvcs.org/wiki/download)有詳細的教學, 這裡有[官方安裝](http://wiki.rabbitvcs.org/wiki/install/ubuntu)

加入 repository

	sudo add-apt-repository ppa:rabbitvcs/ppa
	sudo apt-get update

安裝套件

	sudo apt-get install rabbitvcs-nautilus3 rabbitvcs-thunar rabbitvcs-gedit rabbitvcs-cli

安裝完，打入

	nautilus -q 

重啟資料夾即可

安裝 roboto font 
-----------------
下載 [Roboto Font](http://www.droidforums.net/img/roboto-fonts.zip)

	mkdir ~/.fonts
	sudo unzip roboto-fonts.zip -d ~/.fonts
	sudo fc-cache -f -v

安裝 Droid font
-----------------
	sudo add-apt-repository 'deb http://ppa.launchpad.net/fonts/ubuntu intrepid main'
	sudo apt-get install ttf-droid

安裝 wine
-----------------
	sudo add-apt-repository ppa:ubuntu-wine/ppa
	
安裝 JDownloader
-----------------
	sudo add-apt-repository ppa:jd-team/jdownloader

安裝 QEMU
-----------------
	sudo apt-get install kvm qemu-kvm bridge-utils libvirt-bin virtinst vtun virt-manager

or

	sudo apt-get install qemu-system

使用的方式為 `qemu-system-i386` or `qemu-system-x86_64`，端看你系統是哪種，如

	qemu-system-i386 -fda boot.img
	
如果出現 `failed to find romfile "pxe-rtl8139.bin`，就必須要安裝

	sudo apt-get install kvm-pxe

chmod 的用法
-----------------
chmod (a代表所有人，後面的加號是"增加屬性"，x代表執行)

	chmod a+x sh01.sh

-R代表遞迴整個目錄

	chmod 755 -R /home/vbird

刪除整個目錄(包含裡面檔案)
-----------------

	rm -r dir-name

查看系統是32位還是64位
-----------------

	getconf LONG_BIT

還原compiz桌面
-----------------

	gconftool-2 --recursive-unset /apps/compiz-1

查看內核版本
-----------------

	uname -a

安裝中文的 man
-----------------

	sudo apt-get install manpages-zh

解壓縮
-----------------

	tar -zxvf /tmp/etc.tar.gz

執行shell script
-----------------
	sh sh01.sh

或是把該檔案加入執行屬性

	chmod a+x sh01.sh

chmod(a代表所有人，後面的加號是"增加屬性"，x代表執行)

執行 shell script，並顯示每行的結果
-----------------
	sh -x sh01.sh

還原被改壞的 compiz
-----------------
還原被改壞的`compiz`在Ubuntu11.04中還原Compiz的話會與其它舊版本的還原方式不一樣，
原因在於其配置文件是存放在`/apps/compiz-1`而不是像以前的`/apps/compiz`。（P。S儘量不要這麼做，這個命令有風險）

	gconftool-2 --recursive-unset /apps/compiz-1
	
若是桌面壞了(非gnome桌面)

	unity --reset

顯示目前目錄
---------------

	pwd

使用 Evernote
---------------
只有 nevernotes 可以替代，下載 `nevernote.deb` 之後安裝即可 
 
如果要在 wine下面，使用evernote時，在安裝 wine 之後，要進入terminal之後，打入 

	wine wine /home/eric/下載/Evernote_4.2.1.3716.exe

代表安裝window程式

切換桌面
---------------
	Ctrl + Alt + right arrow

改變 printk 顯示訊息層級
----------
	echo 7 /proc/sys/kernel/printk
