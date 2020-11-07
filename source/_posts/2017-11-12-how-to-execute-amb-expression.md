title: 執行 amb expression
date: 2017-11-12 01:13:59
tags: sicp
toc: true
---

建立 amb 環境
----------------------

先去這邊下載

	https://mitpress.mit.edu/sicp/code/

amb 是 chapter 4 的, 所以選以下這個下載

	ch4-ambeval.scm 	Amb Evaluator (section 4.3)

然後再scheme 中執行, 會發現 load 不進去另一個檔, 其實把兩個檔案合起來就可以了, 存成一個檔案後, 就可以使用指令 load 出來, 如下

	( load "e:\\a.scm") 

`PS：按CTRL-Y 可以貼上文字`
<!--more-->
 
然後按`Ctrl + x`, 再按`ctrl + e`就可以執行 LISP

如果出現了 

    ;Loading "e:\\a.scm"... done
    ;Value: amb-evaluator-loaded

	
就代表載入 成功, 執行amb的部分可以參考以下

	http://uents.hatenablog.com/entry/sicp/059-amb-operator-with-call-cc.md

接下來建立 env, 使用以下指令

    (define the-global-environment (setup-environment))

	
執行 amb 
-------------------------

接下來就可以輸入 

	(driver-loop)

如果出現

    ;;; Amb-Eval input:

此時就是代表進入到 amb 執行器, 輸入

    (amb 1 3 5 )
	
會出現1 , 輸入 

    try-again
	
會出現3

觀察裡面的值
-------------------
例如, 我想觀察變數`exp`, 就可以在 code 中插入

	(newline)
	(display "=== my print ==")
	(newline)
	(display exp)

即可 trace code


心得
------------------

1. 先把輸入參數, 利用 map +analyze 分析完成後, 放到 cprocs 中
2. 再利用 (try-next cprocs), 把每個東西都拿出來


