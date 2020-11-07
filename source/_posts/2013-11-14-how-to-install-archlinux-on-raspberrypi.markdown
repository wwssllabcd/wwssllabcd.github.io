---
layout: post
title: "Raspberry Pi 安裝 Archlinux"
date: 2013-11-14 19:48
comments: true
tags: [How-To, Raspberry Pi]
---

簡短的紀錄一下 Raspberry pi 如何安裝 Archlinux，並且使用 pacman 安裝幾個套件  

一樣去下載頁面下載 arch linux OS 回來後，解壓縮可以得到 img 檔，
安裝完後，開機，使用使用 pietty(talnet 軟體) 連線後登入，此時的安裝方式都與 wheezy 相同  

更新 pacman
================
要先更新 pacman ，否則某些套件會找不到

輸入以下指令

	pacman -Syy
	pacman -Syu

<!--more-->

安裝 transmission 
===============
使用 pacman 安裝，輸入

	pacman -S transmission-cli

可參考[archlinux 的 Transmission 安裝教學](https://wiki.archlinux.org/index.php/Transmission)

安裝完後，先啟動, 讓他產生參數

	systemctl start transmission.service

再停止，以便修改參數

	systemctl stop transmission.service

檢查 status 是否正確停止

	systemctl status |grep transmission

輸入

	transmission-daemon
	
不知為何，要打入這個才會出現.config 的 folder

修改參數後重新啟動

	systemctl daemon-reload
	systemctl restart unit


安裝 samba 
==========
使用 pacman 安裝套件

	pacman -S samba

進入 `/etc/samba/`，把 `smb.conf.default` cp一份為 `smb.conf`，並且修改 `smb.conf`
修改conf這邊不再贅述，若不懂的話，請參閱本站 samba 設定


跑不起來的話, 看看 samba 狀態

	systemctl status samba 

如果出現

	> I disabled starting "samba" and enabled only smbd.service and nmbd.service.
	> Now I can see Raspberry in my second computer, but I cant connect to it, 
	> it fails with "you have no permission to connect"

代表你需要帳號密碼

samba 需要帳號密碼存取

	smbpasswd -a pi

其中pi是要已經存在的帳號，這樣一來應該就可以成功啟動 samba 

