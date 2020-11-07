---
layout: post
title: "簡易 GDB Script 教學，應用"
date: 2013-03-25 19:27
comments: true
tags: [How-To, GDB, GDB Script]
---

緣由
----------------
在 Debug 電腦開機階段時，利用 QEMU 停住 OS, 在遠端的 GDB 每次都要輸入

	set architecture i8086
	target remote localhost:1234 
	b *0x7C00
	
一兩次到還好，若是有時候常常輸入真的就太累了, 而 GDB 可以以 script 的方式擴充指令，就可以減少了負擔了。  
首先我們開一個空白文件，取名叫做`myGdbScript.txt`( myGdbScript.txt 以下稱做GdbScript)

<!--more-->

當作批次檔
----------------
直接在文件中輸入

	set architecture i8086
	target remote localhost:1234 
	b *0x7C00

存檔後，在 terminal 輸入 gdb，進入 GDB 後，輸入以下指令(一樣可以用tab快速輸入)

	source myGdbScript.txt

就會發現他就像批次檔一樣，自動的執行裡面的指令，這樣就不用輸入一堆東西了

自行定義指令
----------------
以上的方法只能執行一個命令，若想要執行多種命令的話也可使用定義指令的方式，在 GdbScript 中輸入

	define local_connect
		set architecture i8086
		target remote localhost:1234
		b *0x7C00
	end

把這串指令叫做`local_connect'
然後在 GDB 中，先把 GdbScript 讀進來

	source myGdbScript.txt

之後就可以打

	local_connect

你就會發現，他執行 define 裡面的三條指令

取得 current intstruction
----------------------
使用 x/5i $cs*16 + $eip 來顯示目前 CS:IP所對應的指令，如下所示

	define cur_intstruction
		x/5i $cs*16 + $eip
	end

 
定義變數
-----------------
在 GDB 中，也是可以定義變數的，如在GdbScript中輸入

	set $ADDRESS_MASK = 0x1FFFFF

	
印出符號
----------------
在script中，輸入如下所示

	printf "---------------------------[ CODE ]----\n"


取得 cpu register的值
---------------------
如下所示，可以把某個register的值設定給變數後，印出來

	define compute_regs
		set $rax = ((unsigned long)$eax & 0xFFFF)
		set $rbx = ((unsigned long)$ebx & 0xFFF
		
		printf "---------------------------[ REG ]----\n"
		printf "AX: %04X BX: %04X ", $rax, $rbx
		printf "\n"
	end
		

Ref: [Remote debugging of real mode code with gdb](http://ternet.fr/wiki/doku.php?id=blog:gdb_real_mode)
		
		
  