title: makefile 心得、教學
date: 2016-10-03 01:13:59
tags: C++, Makefile
toc: true
---

要寫 makefile 之前，首先我們必須要先從最基本的 GCC 編譯指令開始學起，才可以一步一步地建立起 makefile，幸好這不會花我們太多時間


使用 GCC 編譯命令，並且印出 hello world 
============
建立一個檔案叫做 main.c 後，輸入以下指令

	#include<stdio.h>
	int main(){
        printf("\r\nHello World");
	}

接著在命令列(windows 系統可以使用 cygwin)中，輸入 compile 指令

     gcc -c main.c

執行完後會產生 obj file，如 main.o  
而上述所使用的編譯參數如下所示

	-c : 只編譯不連結 

執行連結，使用 gcc -o 指令
     
	 gcc -o test main.o
	 
-o 代表作 link，-o filename 為指定輸出檔名  
此時應該會出現一個叫 test 的檔案  
執行 test 

	./test

	Hello World

編譯時，如果有 header file 的時候，可以使用 -I 參數，如下所示  
這邊是指定 header file 是在哪個目錄可以找的到

     gcc -c -I ./inc main.c

<!--more-->
	 
使用 makefile 簡化
============
建立一個文字檔，取名為 makefile，內容填入如下所示
	
	test: main.o
		gcc -o main.o test
	main.o: main.c
		gcc -c main.c
	clean:
		rm -f *.o *.exe

先解說以下這行
	
	main.o: main.c

main.o: 分號前面代表目標，而後面的 main.c 就是告訴 make，要完成前面那個目標的話，必須要有`main.c`這個前置條件，
所以如果輸入`make main.o` 的話，是會自動執行 `gcc -c main.c` 這段指令的，因為前置條件已經滿足，所以可以執行
執行完就是產生 main.o 了

再看看第一行
	
	test: main.o
	
test: 代表目標，而後面的 main.o 是前置條件，
執行 make test 的時候，make 會找看看有沒有 main.o 這個檔，如果有的話就會執行，如果沒有的話，他會去找看看tag有沒有產生 main.o 的方法，並且嘗試產生出main.o


執行 makefile
============
在 console 輸入 make test 之後，make 會去找有無 test 這個 tag，
這邊有 test 的 tag，而執行 test 的前置條件是必須要有 main.o ，
則 make 會檢查有無 main.o 這個檔案，如果沒有的話，會自動搜尋 makefile 中，有無 main.o 這個檔案的產生方法，

這邊是有的，不過產生 main.o 的先決條件是要有 main.c ，
則 make 會檢查有無 main.c 這個檔案，目前是有的，

所以 make 會先去執行 main.o 那個 tag，也就是 gcc -c main.c，
執行這行指令後，會產生 main.o 這個檔案出來，
所以執行 test 這個 tag 的條件也已經滿足了，所以可以執行 test 這個 tag ，也就是執行 gcc -o main.o test，
所以產生出 test 這個檔案出來了


多個檔案的 makefile
============
加入第二個 .c file 

	main.o: main.c
		gcc -c  main.c
	DataIn.o: DataIn.cpp
		gcc -c DataIn.cpp

可以觀察到 main.o 與 dataIn.o 其實差不多格式，所以應該要有個萬用的格式例如 *.* 這種東西來簡化，
而 make 的確是有這種簡化指令的，他是使用 % 來簡化，但 % 是屬於一對一的，也就是 foo.o 對應到 foo.c，這跟 * 不太一樣，
而目標與前置條件都有萬用符號後，其實 gcc -c 要接的那個檔案名稱，也必須要是一種變數才行

	%.o: %.c
		gcc -c $<

$< : 屬於第一條件，也就是 foo.c  
$@ : 屬於目標條件，也就是 foo.o  

撰寫 makefile 的一些心得
============
先把ld 需要的 object 建立起來，如建立起 obj_files 

     OBJ_FILES = \
        $(OBJDIR)/head.o  \
        $(OBJ_LIB) \
        $(OBJ_KERNEL) \

然後利用 make 的前置規則讓他去找自動產生編譯需求

     system.bin:  $(OBJ_FILES )

再利用萬用符號，讓每個檔案被編譯出來，如下所示

	# == rule for kernel/ ==
	$(DIR_KERENL)/%.o: $(DIR_KERENL) /%.asm
		$(AS) $< -o $@
       
	$(DIR_KERENL)/%.o: $(DIR_KERENL) /%.c
        $(CC) $(CFLAGS) $< -o $@ 

編譯前的 pre-task 
============
make 的前置條件，不見得是一個檔案，也可以是某個 tag

	all: clean mkdir boot.img 
	
這邊代表需要先執行 clean 這個規則，再需要 執行 mkdir 這個規則，然後執行boot.img

make 時不顯示指令
============
在命令前面加上 @ ，代表不顯示該命令，如下所示
@mkdir -p $@


.PHONY 符號的用法
============
例如有時候都會見到
	
	.PHONY: clean

.PHONY。這個符號的目的是告訴 make，"clean" 不是一個真正的檔案目標，只是一個標記，不要把他當成檔案來處理，
避免有檔案真的叫 clean 時，make 會在依賴性判斷時判斷錯誤，那就糗了。


在 make file 中使用 awk
============

	awk '{ print $$1" "$$3 }' system.nm > system.bsb

只有使用一個$的話，會被make吃掉，使用兩個$，就不會消失

makefile 中，定義變數
============
可利用 $(MACRO) 或 ${MACRO} 來存取已定義的變數

M=$(PWD) 表明然後返回到當前目錄繼續讀入、執行當前的 Makefile。

"?= 語法" 
============
?= 語法：
?= 是一個簡化的語法：若變數未定義，則替它指定新的值。否則，採用原有的值。例：
FOO ?= bar
若 FOO 未定義，則 FOO = bar；若 FOO 已定義，則 FOO 的值維持不變。

:= 語法
============
:= 語法
注意到，make 會將整個 Makefile 展開後，再決定變數的值。也就是說，變數的值將會是整個 Mackfile 中最後被指定的值。例：

	x = foo
	y = $(x) bar
	x = xyz	
	# 此時 y 的值為 xyz bar
	
在上例中，y 的值將會是 xyz bar，而不是 foo bar。
您可以利用 := 來避開這個問題。:= 表示變數的值決定於它在 Makefile 中的位置，而不是整個 Makefile 展開後最終的值。


巢狀 make
============
也就是說 make 可以執行其他的 make ，
如每個目錄都有自己的 make ，根目錄的 make 是可以進入到 其他目錄中，跳去執行其他的 make 後再回來
使用 -C 參數。後面帶目錄名稱如下所示

	make -C boot

你就會看到 make 會 Entering directory 後，再做 make 


makefile 建立目錄
============
必須要一個 target來幫助，如 directories，如下所示

	OBJDIR = ./obj
	$(OBJDIR):
		mkdir -p $@
    
	makeDir: ${OBJDIR}

接下來就是在 make all 那邊，加入dependence
	
	all: makeDir 

make 內部變數
============
$?：代表已被更新的 dependencies 的值，也就是 dependencies 中，比 targets 還新的值。    
$@：代表 targets 的值。$<：代表第一個 dependencies 的值    
$* :代表 targets 所指定的檔案，但不包含副檔名    

