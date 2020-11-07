---
layout: post
title: "使用 Ruby 產生 RSS feeds"
date: 2012-10-26 16:28
comments: true
tags: [How-To, Ruby, RSS] 
---

Ruby 是一種很適合拿來處理網路資料的一種語言，
使用 Ruby 網頁框架 ROR or sinatra 都可以很容易的建立動態網頁，
就連產生 RSS feeds 也有專屬的 class  

使用方式
--------------
在檔案前面加入  
```
	require 'rss/maker'
```
	
使用 RSS::Maker 產生 content，如下所示 ( 以下為 code 的一部分 )  

<!--more-->
```
	require 'rss/maker'
	
	strRssTitle = "RSS title"
	aryRssLink  = Array.new
	aryTitle    = Array.new
	aryLink     = Array.new
	aryFullText = Array.new
	aryTime     = Array.new
	aryMedia    = Array.new
	
	content = RSS::Maker.make(version) do |m|
		m.channel.title = strRssTitle
		m.channel.link ="http://myRssWebSite.tw/MyRSS"
		m.channel.description ="By Eric" 

		cnt = aryTxt.length
		cnt.times{ |idx|
			i = m.items.new_item
			i.title = aryTitle[idx]
			i.link = aryLink[idx]
			
			strMedia = ""
			if aryMedia[idx] != ""
				strMedia = "<img src=" + aryMedia[idx] + ">"
			end
				
			i.description = strMedia + aryTxt[idx]
			i.date = aryTime[idx]
		}
	end  
	
	"#{content}"
```	



不過這邊要注意的是圖片的地方，由於 Maker.make 好像沒有提供圖片的屬性，
所以要放圖片，則必須自己組合好 &lt;img src&gt; 後，塞到 description 裡面  

顯示 RSS 的 Title  
```
	m.channel.title = strRssTitle
```

顯示 RSS 的 link，通常是本身 rss feeds 的 網址  
```
	m.channel.link ="http://myRssWebSite.tw/MyRSS"
```
	
對於該 RSS 的 描述  
```
	m.channel.description ="By Eric" 
```
	
使用 m.items.new_item 來新增 RSS item，
不過你的 RSS 裡面應該不止一個 item，所以最好是用一個迴圈來做  
```
	i = m.items.new_item
```

這邊 RSS item 有好幾種屬性，包括 title, link 等
```
	i.title = aryTitle[idx]
	i.link = aryLink[idx]
	i.description = strMedia + aryTxt[idx]
	i.date = aryTime[idx]
```

最後再輸出 content 就完工了
```
	"#{content}"
```
	
驗證
------------
ok，產生完了，就要來做驗證了，
W3 提供了驗證 RSS feeds 的服務，可以到 [w3c](http://validator.w3.org/feed/) 的網站，
把你的 link 貼上去就可以幫你驗證了。



	
後記：
----------------
這個 code 其實是要做成一個 function，並接受一個 rss items 的 collection 當作參數，
不過可能要等我有空的時候才會修嚕


參考資料：
---------------------
[使用Ruby on Rails 解析及创建RSS](http://cpccai.iteye.com/blog/137941)  
[簡單產生 RSS Feeds 及簡易部落格聯播功能](http://ithelp.ithome.com.tw/question/10011541?tag=rt.rq)  

