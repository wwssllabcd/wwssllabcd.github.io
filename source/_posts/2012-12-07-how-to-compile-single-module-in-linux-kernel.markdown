---
layout: post
title: "在linux kernel中，編譯單一模組"
date: 2012-12-07 00:02
comments: true
tags: [How-To, Lunux kernel]
---

在學習linux driver 時總是需要 compile kernel，但是為了單一module而重新compile整個kernel可就浪費時間了
所以最好的方式就是只compile單一module，這樣不但數度快，也方便debug

取得kernel source code 
---------------------
首先你要知道目前你系統的 kernel 的版本號，才可以去取回對應的 kernel source
使用
	uname -r
查詢目前所使用的 kernel 版本是個不錯的選擇

<!--more-->

下載kernel source code
----------------------
安裝 kenrel source 也可以組合成以下的指令
	sudo apt-get install linux-source-$(uname -r)
	
或是先用apt-cache search找
	sudo apt-cache search linux-source 
再利用apt-get安裝
	sudo apt-get install linux-source-2.6.32 
如果很清楚知道那個版本，也可以直接到[The Linux Kernel Archives](https://www.kernel.org/)下載
下載完，解壓縮進行下一步了。


取回的原本的.config & Modules.symvers
------------------------------------
.config 是kernel 的option，如果你compile 單一module是使用不同的option時，則會產生不匹配的問題，
所以最好是使用目前系統使用的option
原本的.config放在 /boot 目錄下，如
	cp /boot/config-2.6.35-generic /home/eric/linux-kernel/.config

copy Modules.symvers 
把  /lib/modules/<your kernel name >/build 下面的  Modules.symvers，copy 到編譯的source 目錄下


開始編譯
-----------------------------------
回到kernel source 最上層目錄，編譯單一內核區塊，如編譯 drives/usb/storage 
	make prepare
	make script
	make M=drives/usb/storage

註：usb_storage.ko是整個 /storage 編譯的結果，並沒有usb_storage.c這種檔案

載入module
-----------------------------------
編譯完成不代表沒事，因為 insmod 可能會出錯，如果是 -1 Invalid module format 的話，可以先看看dmesg是否也有錯誤訊息
打入dmesg，出現 usb_storage: no symbol version for module_layout

也可以使用modinfo指令比較原本的module與目前compile出來的有沒有差異

要檢查很多地方，可根據[insmod: error inserting : -1 Invalid module format 解决办法 ](http://blog.csdn.net/wdove/article/details/6561650)
的檢查方法檢查  

如果要修改vermagic，要在最上層的 makefile那邊改，改完記得要用
make prepare
make script
再做一次

如果還不行，可能就是 Modules.symvers所造成的
把  /lib/modules/<your kernel name >/build 下面的  Modules.symvers，copy 到編譯的source 目錄下，應該就可以解了

若成功insmod後，修改usb.c中的usb_storage_driver中的 .name一項，改成`usb-storage-test`後，compile usb-storage，並且insmod 成功後，
打入dmesg觀察 kernel 訊息，如果有出現 register usb-storage-test的話，代表我們客制化的module成功被載入了

測試方式
--------------------------------
提供我的載入方式：

1.  先拔掉目標usb
2.  使用rmmod卸載之前的module
3.  使用insmod載入
4.  觀察kernel message，是否載入正確
5.  插上usb，觀察module運作


打開 usb debug
---------------------
在drivers/usb/storage/debug.h
不過打開會有WARNNING錯誤，所以就不打開了

