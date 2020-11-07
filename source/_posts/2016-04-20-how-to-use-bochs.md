title: bochs 使用教學，心得
date: 2016-04-20 17:23:25
tags: bochs
toc: true 
---
安裝
============
使用 apt 安裝的是沒有debug 功能的，bochs 通常一出現視窗運行會沒辦法輸入指令，
如果你只是要單純的執行環境的話，就使用 apt安裝吧，如果要 debug 功能的話，就要自行 compile

下載 [bochs 2.6.8 source code](https://sourceforge.net/projects/bochs/files/bochs/2.6.8/)

安裝(可參考 orange's P10)
輸入下列指令 

	tar vxzf bochs-2.6.8.tar.gz
進入目錄後，開始設定

	./configure --enable-disasm  --enable-debugger
	
如果要使用 gdb 的話，就不能用 `--enable-debugger` ，要換成  `--enable-gdbstub`
接下來就是

	make
	sudo make install

<!--more-->
	
Bochs ini 配置
============
配置文件詳解可看 
[The configuration file bochsrc](
http://bochs.sourceforge.net/doc/docbook/user/bochsrc.html)
也可見 linux 內核完全註釋 CH17，
這邊列出我自己的 src 檔


	romimage: file=/usr/share/bochs/BIOS-bochs-latest
	megs: 16
	vgaromimage: file=/usr/share/vgabios/vgabios.bin
	floppya: 1_44=system.img, status=inserted
	ata0-master: type=disk, path="80.img", mode=flat
	boot: a
	log: bochsout.txt
	mouse: enabled=0
	display_library: x
	#debug_symbols: file=system.bsb
	#gdbstub: enabled=1, port=1234

把以上的ini存成 `bochsrc.txt` 就可以了，另外如果有多個 ini 檔要切換時可以使用 `-f` 參數，如下所示

	bochs -f anotherBochIni.txt

bochs 載入 debug symbols 
============
載入symbol 的方是有兩種，一種是手動，一種是ini載入，以下兩者都會介紹到

修改 symbol 格式
--------------------
官方說，bochs的 symbol 的格式為

	The symbol file consists of zero or more lines of the format
	"%x %s"
	
也就是說，只有文字檔格式 "%x %s"  才可以載入，不像GDB可以載入bin，
而這邊可以先觀察一下 nm 輸出的格式如下

     00000000 T startup_32
	 
中間有一個 type，不符合 bochs 規定的格式，所以要把那個 type 行去掉
如果要改變格式，可以利用 awk 來幫助，例如要改變 nm 檔為兩行時，可用下列 awk 指令

     awk '{ print $1" "$3 }' system.nm  > system.bsb

$1與$3 分別代表直排一與直排三，若要搭配在 make 中使用的時候，要加兩個$，如下所示 

	nm:
       nm system.elf |sort > system.nm
       awk '{ print $ $1" "$ $3 }' system.nm > system.bsb

nm 檔要經過修改後，才可以正確的載入到 bochs

使用 ini 載入 debug file 
-------------------
在ini中輸入

	debug_symbols: file="system.bsb"

system.bsb 為你修改過的 symbol file 

Bochs 手動載入 symbol
---------------------------
而使用 ldsym 的時候，要加雙引號，如下所示

     ldsym "system.bsb"

	 
使用 gdb 當作測試 client
-------------------
安裝 bochs 時，必須要打開 `--enable-gdbstub` 後，在 ini 中，加入參數即可

     gdbstub: enabled=1, port=1234
	 
bochs debug 指令 
============
可以在執行時，輸入 h ，會有簡單的指令列表，
而在 bochs 中，下中斷要加雙引號，如下所示

     b "TestA" 
	 
也可以列出變數的值，如

	x/10 "idt"

其實 bochs 應該就是根據 symbol file ，來找出對應的記憶體位置而已，

Bochs 操作蠻像 GDB 的，以下列出常用的 bochs 的debug 指令
	
|   指令         | 說明                                 |
|----------------|--------------------------------------|
| c              | continue，執行 OS                    |
| s              | 單步(會進入function)                 |
| n              | 單步(不進入function)                 |
| b "main"       | 下中斷在 function main 的起始位置，使用時，記得要載入符號表         |
| d              | 刪除中斷                             |
| blist          | 列出所有中斷點                       |
| x/10 addr      | 列出 addr 的位置的值                 |
| q              | 離開 bochs                           |



其他的指令可見 [Using Bochs internal debugger](http://bochs.sourceforge.net/doc/docbook/user/internal-debugger.html) 說明
	 
troubleshooting 
============

fatal error: X11/extensions/Xrandr.h: No such file or directory
---------------------
ANS:  missing libxrandr-dev


display library 'sdl' not available
-----------------------------------
手動安裝的時候，預設的display選項為 x (此例為sdl)
所以`bochssrc.txt`設定要設為

	display_library: x

	
[BIOS ] No bootable device
---------------
有很多問題會造成這個錯誤，這邊只是列舉一個

	ans:  floppya image size doesn't match one of the supported types

OS 若是使用 floppy 模擬的話，磁碟最好寫滿到count 2888。最後加上

	count=2883 seek=5 conv=notrunc

以seek5個sector為例，這邊就是寫2883個sector


Reference:
============
http://www.groad.net/bbs/thread-678-1-1.html    
http://www.cnblogs.com/long123king/p/3568575.html     
