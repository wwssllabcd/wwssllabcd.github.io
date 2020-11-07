---
layout: post
title: "打造一個 Firefox 附加元件"
date: 2014-05-03 01:32
comments: true
tags: [How-To, Firefox extension]
toc: true
---

開始之前
======================
以下會介紹 Mozilla 所推出的 Add-on SDK, 並且用它來打造你第一個 Firefox extension，要做的套件功能是利用 Add-on SDK 抓取匯率網站的資料，重新整理之後顯示出來  

首先先建立開發平台, 以下是需要的工具, 分別是 addon-sdk, python, firefox
![Prerequisites](https://lh6.googleusercontent.com/-z0GbZ6KBNSA/VJexItwKtsI/AAAAAAAAsuM/2tHAspCDwrs/s0/Prerequisites.jpg)

* addon-sdk 請到 [Mozilla 官網下載](https://developer.mozilla.org/en-US/Add-ons/SDK/Tutorials/Installation), 
下載完解壓縮後, 放在你喜歡的地方即可( 如 d:\project\addon-sdk)

* 除了 addon-sdk 外, 也需要 [Python](http://zh.wikipedia.org/zh-tw/Python), 由於我的平台是 windows, 所以就裝了 [Python for win](https://www.python.org/downloads/), 
雖然有 `2.7.6`與 `3.4.0` 可以選擇, 不過mozilla 建議裝 2.7.X 的比較好, 安裝步驟就是下一步一直按下去, 沒難度.

都安裝好了之後, 就使用[命令提示字元](http://zh.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E5%AD%97%E5%85%83)(以下簡稱 command line), 到 addon-sdk 下面
接下來鍵入
	bin\activate
就會看到命令提示字元變得不太一樣了, 如下狀況
![activate](https://lh4.googleusercontent.com/-zx2gUQKNiDA/VJexI1cktEI/AAAAAAAAsuI/Up_TR96b8Hw/s0/activate.jpg)  

這時候就是進入了 addon-sdk 開發模式, 不過這個動作在每一次關掉 command line 就要重複一次, 所以最好的做法還是把這一連串的啟動動作做成批次檔如下(我 addon-sdk 放在 d:\addon-sdk)

	d:
	cd\addon-sdk
	bin\activate

<!--more-->

建立一個新的專案
======================
根據上一步, 啟動了addon-sdk 後, 在你想要的地方開始建立專案資料夾, 並且 initial 他

	mkdir my_addon
	cd my_addon
	cfx init

執行起來大概長這樣   
![cfxInit](https://lh6.googleusercontent.com/--JE6-Q96vNg/VJexJkW8erI/AAAAAAAAsuY/dHz8BMapgsM/s0/cfxInit.jpg)  

開始第一支程式 -- Hello World
==============
目錄中的 lib 的資料夾, 裡面有個 `main.js`, 
則是套件的起點, 用編輯器打開, 鍵入以下程式碼

```
	var widgets = require("sdk/widget").Widget;
	myWedget = widgets({
		id: "I-am-wedget",
		label: "wedget",
		content: "Hello World!",
		tooltip: "I-am-tooltip",
		width: 100
	});
```

把`main.js`編碼選擇 UTF-8 後儲存關閉, 接下來就可以看看效果如何了  
鍵入 

	cfx run

之後就會跳出一個乾淨的 Firefox, 並且上面有你剛做的套件, 基本上 Wedget 算是顯示用的, 很簡單吧
![HelloWorld](https://lh4.googleusercontent.com/-j8rzmz7qQeg/VJexKJ9PyhI/AAAAAAAAsug/oCEDsbIeVnU/s0/helloworld.jpg)  

有關資源的存取
-------------
my_addon 目錄會出現數個資料夾, 如lib, data, doc, test 等, 其中 data 區對應到 code 裡面的 resource 的根目錄,
如`./a.png`的意思就是代表存取 data 區裡面的 a.png 的意思，如果要用到外部的 JavaScript file ，也是放在 ./data中  

放出 Message 
----------------
使用 console.log 即可，如下所示  

	console.log("A= %D", 100)

實戰 -- 製作一個抓取匯率的套件
==================

抓取某銀行網頁匯率的資料, 使用 page-worker 取回網頁
----------------------
此時就要用 [page-worker](https://developer.mozilla.org/en-US/Add-ons/SDK/High-Level_APIs/page-worker) 來幫你實現了, 
延續剛剛的 Wedget, 這次我們來抓某個網頁的資料, 並且用 tooltip 的方式, 把它顯示在 Wedget 上面  

```
	var widgets = require("sdk/widget").Widget;
	myWedget = widgets({
		id: "I-am-wedget",
		label: "wedget",
		content: "Hello World!",
		tooltip: "I-am-tooltip",
		width: 100
	});
	
	var pageWorkers = require("sdk/page-worker");
	var myWedget, myPW
	var script = "postMessage(document.body.innerHTML);";
	var targetURL = "http://rate.bot.com.tw/Pages/Static/UIP003.zh-TW.htm";

	myPW = pageWorkers.Page({
		contentURL: targetURL,
		contentScript: script,
		contentScriptWhen: "ready",
		onMessage: function(message) {
			myWedget.tooltip = message;
		}
	});
```

這邊說明一下 PageWorkers 的屬性部分  

* contentURL: 代表的是要抓的 url
* contentScript: 代表的是,抓回來要做什麼要的處理, 這邊我們只有把 document.body.innerHTML 藉由 postMessage 傳送出去
* onMessage: 則負責接收 postMessage 丟出來的訊息, 也就是該網頁的 innerHTML 了, 並且把他設給 Wedget 的 tooltip 

我隨便選一個提供匯率的網頁, 並且把他設定給 targetURL, 他會把該網頁的 html, 顯示成為 Wedget 的 tooltip 上面, 
原本把滑鼠移上去會顯示 `I-am-tooltip`, 現在已經變成亂七八糟的 html code 了, 
如下所示![htmlCode](https://lh5.googleusercontent.com/-QG7ut0HyUXo/VJexKX3oBrI/AAAAAAAAsuk/qIfCLqA8x50/s0/htmlCode.jpg)  

進一步的清理網頁資料
-----------------
接下來就有點是 dirty work 了, 我們如果要把每個匯率的資料抓出來的話, 要怎做?  
先觀察 html, 發現在匯率前面都會有串字串叫

	"/Images/Flags/America.gif"
	
結束的字串都是為

	"</td><td class="

我們利用這兩個字串當作識別項, 試看看能否把匯率的值給抓出來  

```
	var findKeyWord = function(keyWord, source, cellNo)
	{
		var startWord = "/Images/Flags/America.gif";
		var endWord = "</td><td class=";

		//通常是Table的起始
		var startIdx = source.indexOf(startWord);

		//從table的起點往後找KeyWord
		var keyWordIdx = source.indexOf(keyWord,startIdx);

		//從keyword往後找結束符號
		var temp = keyWordIdx;
		for(i=0; i<cellNo; i++){
			temp = source.indexOf(endWord,temp)+1;
		}
		endIdx = temp-1;

		//從EndIndex往前找，找>關鍵字，但是要往前移一個
		var curIdx = source.lastIndexOf(">",endIdx)+1;
		return source.substring(curIdx,endIdx);
	}
```

傳入要找的關鍵字(keyword), 資料來源(source), 還有要取第幾格的號碼(cellNo)
來取出特定格數的資料出來, 貌似可以做到.  

再加點輔助程式吧, 現在只要呼叫 getResult 並且把 html 傳入的話, 就會得到完整的匯率表了  

```
	var getResult = function(msg){
		var res = "";
		res += "幣別： 現金買入 , 即期買入";
		res += getOneString("美金 ：", "美金 (USD)", msg);
		res += getOneString("港幣 ：", "港幣 (HKD)", msg);
		res += getOneString("英鎊 ：", "英鎊 (GBP)", msg);
		res += getOneString("澳幣 ：", "澳幣 (AUD)", msg);
		res += getOneString("加拿大幣：", "加拿大幣 (CAD)", msg);
		res += getOneString("新加坡幣：", "新加坡幣 (SGD)", msg);
		res += getOneString("新加坡幣：", "瑞士法朗 (CHF)", msg);
		res += getOneString("日圓 ：", "日圓 (JPY)", msg);
		res += getOneString("南非幣 ：", "南非幣 (ZAR)", msg);
		res += getOneString("瑞典幣 ：", "瑞典幣 (SEK)", msg);
		res += getOneString("紐元 ：", "紐元 (NZD)", msg);
		res += getOneString("泰幣 ：", "泰幣 (THB)", msg);
		res += getOneString("菲國比索：", "菲國比索 (PHP)", msg);
		res += getOneString("印尼幣 ：", "印尼幣 (IDR)", msg);
		res += getOneString("歐元 ：", "歐元 (EUR)", msg);
		res += getOneString("韓元 ：", "韓元 (KRW)", msg);
		res += getOneString("越南盾 ：", "越南盾 (VND)", msg);
		res += getOneString("馬來幣 ：", "馬來幣 (MYR)", msg);
		res += getOneString("人民幣 ：", "人民幣 (CNY)", msg);	
		return res;
	}
	
	var getOneString = function(title, keyWord, html){
		var res_1, res_2;
		var res = "";	
		res_1 = findKeyWord(keyWord, html, 2);
		res_2 = findKeyWord(keyWord, html, 4);
		res = "\n" + title + res_1 + ", " + res_2;
		return res;
	}

	var findKeyWord = function(keyWord, source, cellNo)
	{
		var startWord = "/Images/Flags/America.gif";
		var endWord = "</td><td class=";

		//通常是Table的起始
		var startIdx = source.indexOf(startWord);

		//從table的起點往後找KeyWord
		var keyWordIdx = source.indexOf(keyWord,startIdx);

		//從keyword往後找結束符號
		var temp = keyWordIdx;
		for(i=0; i<cellNo; i++){
			temp = source.indexOf(endWord,temp)+1;
		}
		endIdx = temp-1;

		//從EndIndex往前找，找>關鍵字，但是要往前移一個
		var curIdx = source.lastIndexOf(">",endIdx)+1;
		return source.substring(curIdx,endIdx);
	}
```

很髒很麻煩, 且一旦該網頁換掉格式就沒用了, 不是嗎?  
我也沒辦法, 不推 API介接, 不推開放資料統一格式就是這樣麻煩, 先不提了.  
現在我們只要在 pageWorkers 的 onMessage 加上 剛新作的 function -- getResult 就可以了   
```
	onMessage: function(message) {
		myWedget.tooltip = getResult(message);
	}
```

把所有東西組裝起來
-----------------

來看看效果吧, 很簡單吧   
![result](https://lh6.googleusercontent.com/-XLs50aKm4qc/VJexK0N9uwI/AAAAAAAAsuw/YTqSzO1Pad0/s0/result.jpg)  

我再把整個程式整理一下,大家可以參考  

```
	var widgets = require("sdk/widget").Widget;
	var pageWorkers = require("sdk/page-worker");
	var myWedget, myPW

	myWedget = widgets({
		id: "erw-wedget",
		label: "erw",
		content: "即時匯率",
		tooltip: "wait ready",
		width: 50
	});
  

	var script = "postMessage(document.body.innerHTML);";
	var targetURL = "http://rate.bot.com.tw/Pages/Static/UIP003.zh-TW.htm";
  

	myPW = pageWorkers.Page({
		contentURL: targetURL,
		contentScript: script,
		contentScriptWhen: "ready",
		onMessage: function(message) {
			myWedget.tooltip = getResult(message);
		}
	});

	var getResult = function(msg){
		var res = "";
		res += "幣別： 現金買入 , 即期買入";
		res += getOneString("美金 ：", "美金 (USD)", msg);
		res += getOneString("港幣 ：", "港幣 (HKD)", msg);
		res += getOneString("英鎊 ：", "英鎊 (GBP)", msg);
		res += getOneString("澳幣 ：", "澳幣 (AUD)", msg);
		res += getOneString("加拿大幣：", "加拿大幣 (CAD)", msg);
		res += getOneString("新加坡幣：", "新加坡幣 (SGD)", msg);
		res += getOneString("新加坡幣：", "瑞士法朗 (CHF)", msg);
		res += getOneString("日圓 ：", "日圓 (JPY)", msg);
		res += getOneString("南非幣 ：", "南非幣 (ZAR)", msg);
		res += getOneString("瑞典幣 ：", "瑞典幣 (SEK)", msg);
		res += getOneString("紐元 ：", "紐元 (NZD)", msg);
		res += getOneString("泰幣 ：", "泰幣 (THB)", msg);
		res += getOneString("菲國比索：", "菲國比索 (PHP)", msg);
		res += getOneString("印尼幣 ：", "印尼幣 (IDR)", msg);
		res += getOneString("歐元 ：", "歐元 (EUR)", msg);
		res += getOneString("韓元 ：", "韓元 (KRW)", msg);
		res += getOneString("越南盾 ：", "越南盾 (VND)", msg);
		res += getOneString("馬來幣 ：", "馬來幣 (MYR)", msg);
		res += getOneString("人民幣 ：", "人民幣 (CNY)", msg);	
		return res;
	}

	var getOneString = function(title, keyWord, html){
		var res_1, res_2;
		var res = "";	
		res_1 = findKeyWord(keyWord, html, 2);
		res_2 = findKeyWord(keyWord, html, 4);
		res = "\n" + title + res_1 + ", " + res_2;
		return res;
	}

	var findKeyWord = function(keyWord, source, cellNo){
		var startWord = "/Images/Flags/America.gif";
		var endWord = "</td><td class=";

		//通常是Table的起始
		var startIdx = source.indexOf(startWord);

		//從table的起點往後找KeyWord
		var keyWordIdx = source.indexOf(keyWord,startIdx);

		//從keyword往後找結束符號
		var temp = keyWordIdx;
		for(i=0; i<cellNo; i++){
			temp = source.indexOf(endWord,temp)+1;
		}
		endIdx = temp-1;

		//從EndIndex往前找，找>關鍵字，但是要往前移一個
		var curIdx = source.lastIndexOf(">",endIdx)+1;
		return source.substring(curIdx,endIdx);
	}
```

後記
==============
其實 Mozilla 也有出[官方教學](https://developer.mozilla.org/en-US/Add-ons/SDK/Tutorials/Installation), 大家也可以參考看看
	







