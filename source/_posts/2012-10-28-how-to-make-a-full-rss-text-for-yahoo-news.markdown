---
layout: post
title: " 製作 Yahoo news 全文輸出的 RSS"
date: 2012-10-28 09:37
comments: true
tags: [How-To, Ruby, Nokogiri, RSS] 
---

緣由
---------------
每次使用 Yahoo News 的 RSS 訂閱功能都有一種美中不足的感覺，就是他只有提供部分文章，而且是少的可憐，
如果你用 Flipboard 之類的訂閱軟體來做閱讀的話，還真的會很不舒服，所以就讓我們來打造一個全文輸出的 RSS Feeds 吧。  

整個想法的 flow 如下

1. 取回 yahoo news RSS 的資料，擷取每則新聞的連結
2. 根據 Rss 的連結，再把每則新聞的內文取回來
3. 把所有內文取回來後，產生新的 RSS 輸出

工具
---------------
我使用的是 ruby + sinatra(有點類似小型的 ROR) + Nokogiri，所以要先搞定這三個，
不過安裝的方式也只是用 gem install 就搞定了，沒啥難度。   

因為以下的教學是假定你對這兩種工具有一定的認識，所以如果你對 sinatra 與 Nokogiri 不熟的人可能要先去熟悉一下。
若對 nokogiri 不熟的朋友們，這邊有篇 [Nokogiri 教學、簡介](http://wwssllabcd.github.com/blog/2012/10/25/how-to-use-nokogiri/) 可以參考看看。  

<!--more-->

取回 yahoo news RSS 的資料，擷取每則新聞的連結
---------------

```
	# encoding: utf-8
	
	require 'open-uri'
	require 'nokogiri'
	
	url = "http://tw.news.yahoo.com/rss/realtime"
	doc = Nokogiri::XML( open( url ) )
```

儲存 RSS 所提供的資料
---------------
RSS 的 doc 取回來之後，就必須要取每篇新聞的標題，日期，還有最重要的該篇新聞的外部連結
建立一個簡單的 class 用來對應到 Rss 中，每個新聞的 Item  

```
	class MyRssItem
		attr_accessor :title
		attr_accessor :pubData
		attr_accessor :link
		attr_accessor :text # 放置全文地方
	end

	aryRssItem = Array.new
	
	doc.xpath('//item').each do |node|
		ri = MyRssItem.new
		ri.title = node.xpath('title').text
		ri.link = node.xpath('link').text
		ri.pubData = node.xpath('pubDate').text
		aryRssItem.push(ri)
	end
```

把每則新聞的內文取回
--------------
先根據 rss 取回來的 link，取回每篇 doc，放在 aryFullDoc 中

```
	aryFullDoc = Array.new
	aryRssItem.each{ |item|
		fullDoc = Nokogiri::XML( open( item.link ) )
		aryFullDoc.push(fullDoc)
	}
```

收集完所有的 FullDoc 後，就針對每個 doc 做解析，取出全文後，把全文放在 MyRssItem 中的 text 

```
	aryFullDoc.size.times{ |i|
		tmpDoc = aryFullDoc[i]
		txt = tmpDoc.xpath('//div[@class="yom-mod yom-art-content "]').to_html
		aryRssItem[i].text = txt
	}
```

產生新的 RSS 輸出
-----------------
一切準備就緒後，就要開始產生 RSS Feeds
這邊先建立一個 sub function，傳入 aryRssItem 後會自動產生 content

```
	def	genRss( strRssTitle, aryRssItem)
		version ="2.0"
		content = RSS::Maker.make(version) do |m|
			m.channel.title = strRssTitle
			m.channel.link ="https://apps-ericwang.herokuapp.com/YahooFullRSS"
			m.channel.description ="By Eric" 
			aryRssItem.each{ |item|
				i = m.items.new_item
				i.title = item.title
				i.link = item.link
				i.date = item.pubData
				i.description = item.text
			}
		end  
	end
```

對於 Rss feeds 產生方式不太瞭解的，可以看 [使用 Ruby 產生 RSS Feeds](http://wwssllabcd.github.com/blog/2012/10/26/how-to-generate-rss-feeds-by-ruby/) 這篇教學

接下來就是輸出了，sinatra 輸出是直接在網頁上輸出的方式為 #{variable}

```
	rss = genRss(strTitle, aryRssItem)
	"#{rss}"
```

這樣就全部完工了

結論
---------------------
使用 Ruby + sinatra + nokogiri 做網頁擷取實在是很方便，很輕鬆就可以完成以前很難做到的事情  

不過實際上放到一般的 RSS reader 上，疑，有點慢喔，如果把這個 Feeds 放在網路上，實際的讓他存取看看，你會發現
連 W3C 的規格都過不了，每個 link 取回 doc 的時間差不多要 1 sec，10 個 link 共要 10 秒，難怪會 time out 
看來還要再改造一番，不過這又是後話了，等我有空的時候再寫吧



