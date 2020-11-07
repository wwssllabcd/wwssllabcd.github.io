title: Firefox addon-SDK 介紹, page-mod
date: 2014-12-21 16:45:54
tags: [How-To, Firefox extension]
toc: true
---

使用時機
================
今天要介紹的是 Firefox addon-SDK 中的 [page-mod][lnk-PageMod], 使用時機為`想要把讀回來的網頁, 再加以修改`, 
例如

1. 把某網站的排版重新排一下, 例如台鐵的火車時刻表
2. 想把某些網站廣告區塊移除掉
3. 想把網頁中特定資訊給顯現出來, 例如某影音網站的影片下載連結

就可以使用這個, 使用的方式如下

page-mode 簡介
================
官方給的範例如下  

	var pageMod = require("sdk/page-mod");
	pageMod.PageMod({
		include: "*.mozilla.org",
		contentScript: 'window.alert("Page matches ruleset");'
	});

<!-- more -->
這邊簡單的說明使用方式  

	include: 要對哪個 url 做動作, 符合的才會做接下來的動作  
	contentScript: 符合include 的條件後, 接下來的動作

以上程式碼意思為假若 url 是 `*.mozilla.org` 的話, 則會跳出訊息`Page matches ruleset`, 
當然簡單的 JS 可以這樣弄, 比較複雜的 JS 建議使用 file, 如下所示

	pageMod.PageMod({
		include: "*.mozilla.org",
		contentScriptFile: "./my-script.js"
	});
	
這邊從 `contentScript` 變成了 `contentScriptFile`, 然後 `./my-script.js` 的路徑是在 `/data` 區, 來看看`my-script.js`裡面長怎樣吧   
	
	//my-script.js
	window.alert("Page matches ruleset");   

只有一行, 其實就是把 contentScript 中的括號拿掉, 就是.js中的內容了
	
	
實戰 -- 製作某影音網站的下載器
================
先來看一下 main.js, 當然, 目標網站的網址被我馬賽克掉了:D    

	var pageMod = require("sdk/page-mod");
	var targeURL = "http://www.thisXX.com/video/*"
	pageMod.PageMod({
		include: targeURL,
		contentScriptFile: "./my-script.js"
	});

main.js 一如往常的簡單, 這邊說明了要針對哪個網站做動作外就沒事了, 一樣的我們把難的工作交給外部的 Java Script 檔案  

找出下載連結
--------------
我們的目標就是該網頁的唯一的 flv 連結 -- `即 http 開頭, .flv結尾`, 先做一個 function 來取連結

	var getFlvUrl = function(source){
		// find .flv
		var endWord = ".flv";
		var endPtr = source.indexOf(endWord);
		var startIdx = source.lastIndexOf("http://", endPtr);
		var len = endWord.length;
		return source.substring(startIdx, endPtr + len  );
	}
	
這邊使用 indexOf 來找連結, 事實上用 regualr express 應該會更簡單, 之後再改善即可  

在原網頁中, 加入下載連結
--------------
接下來就是找一個地方放這個連結, 但如果隨便放的話, 有可能或破壞原始網頁的結構, 所以最好網頁還是一樣維持原狀, 只是在某個地方偷偷插入一段下載連結, 
先來做一個超連結的 html code, 然後再把這段 code 插入原來的網頁

	var downloadInfo = "<P><a href=" + url + ">" + url + "</a><P>";

接下來就是製作插入的 function, 這邊的概念也很簡單, 給定一個插入的目標還有字串, 先利用keyword 找出插入的點後, 利用substring 把原始網頁一分為二,
接下來就是返還修改過的網頁, 一整個超簡單  
	
	var insertString = function(source, msg, keyword){
		var idx = source.indexOf(keyword) + keyword.length;
		var front = source.substring(0, idx);
		var back =  source.substring(idx, source.length);	
		return front + msg + back;
	}

把所有東西組裝起來
-----------------
最後就是把工具都組合起來, 這邊採用`點擊`兩字當作插入的關鍵字位置, 利用 `getFlvUrl(`) 取出 url, 
利用 `insertString()` 插入下載連結後, 返還修改過的 html code 給瀏覽器    

	var url = getFlvUrl(document.body.innerHTML); 
	var insert = "<P><a href=" + url + ">" + url + "</a><P>";
	var html = insertString(document.body.innerHTML, insert, "點擊");
	document.body.innerHTML = html;

整個`my-script.js`如下所示
	
	//my-script.js
	var getFlvUrl = function(source){
		// find .flv
		var endWord = ".flv";
		var endPtr = source.indexOf(endWord);
		var startIdx = source.lastIndexOf("http://", endPtr);
		var len = endWord.length;
		return source.substring(startIdx, endPtr + len  );
	}

	var insertString = function(source, msg, keyword){
		var idx = source.indexOf(keyword) + keyword.length;
		var front = source.substring(0, idx);
		var back =  source.substring(idx, source.length);
		return front + msg + back;
	}

	var url = getFlvUrl(document.body.innerHTML); 
	var insert = "<P><a href=" + url + ">" + url + "</a><P>";
	var html = insertString(document.body.innerHTML, insert, "點擊");
	document.body.innerHTML = html;

結語
================
Firefox 的 addon-SDK 其實把很多事情都簡化了, 做一個附加元件其實很簡單的


[lnk-PageMod]: https://developer.mozilla.org/en-US/Add-ons/SDK/High-Level_APIs/page-mod
