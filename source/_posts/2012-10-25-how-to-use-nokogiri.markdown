---
layout: post
title: "Nokogiri 教學、簡介"
date: 2012-10-25 18:09
comments: true
tags: [How-To, Ruby, Nokogiri] 
toc: true
---

Nokogiri 是 Ruby 上的一個 HTML, XML, SAX 的 parser，他可以藉由 `XPath` or `CSS3 selectors` 來尋找 `XML/HTML` 中的 tag，功能強大，速度快速  

安裝
==============
使用 gem 安裝

	gem install nokogiri

使用時，要在 .rb 檔案 include nokogiri 與 open-uri

	require 'nokogiri'
	require 'open-uri'


<!--more-->

範例
==============

取回目標的 Document
---------------------------
有好幾種方式, 從檔案, 或是從 url  
從檔案  

	html_doc = Nokogiri::HTML( File.open("test.html") )
	xml_doc  = Nokogiri::XML( File.open("test.xml") )

從 url  

	url = 'http://www.google.com/search?q=sparklemotion'
	doc = Nokogiri::HTML(open( url ))

建立一個測試的 html
-------------------

當然我們可以用上述的方式取得某網頁的 doc，不過為了教學起見，這邊我們建立一個簡單的 html，來測試 nokogiri

```
	require 'nokogiri'
	require 'open-uri'
	
	htmlData = "
	<html>
		<title> This is a simple html </title>
		<body id='story_body'>
			<h2> this is h2 in story_body </h2>
		</body>
		<h1> test h1-1 </h1>
		<h1> test h1-2 </h1>
		<h3>
			<img src = 'goodPic-1.jpg' >
			<a href = 'www.google.com'> google web site </a>
			<img src = 'goodPic-2.jpg' > 
			<a href = 'www.yahoo.com'> yahoo web site </a>
		</h3>
		<div class= 'div_1'>
			<h2> this is h1 in div_1 </h2>
		</div>
	</html>
	"
	doc = Nokogiri::HTML( htmlData )
```
		
使用 Nokogiri::HTML( htmlData ) 取回 doc 之後，就可以來玩一些有趣的事了
		
找出所有 < h1 > 標籤的值
---------------------------

	p doc.xpath("//h1")   

result:

	<h1>test H1-1</h1>
	<h1>test H1-2</h1>

其中 `//` 線代表"全部"的意思，使用 `//` 回傳的是一個集合體，所以可以用陣列的方式取值  

	p doc.xpath("//h1")[0]
	p doc.xpath("//h1")[1]
	p doc.xpath("//h1")[2]

result:

	<h1>test H1-1</h1>
	<h1>test H1-2</h1>
		
這邊要注意的是，如果沒有該陣列，會回傳空值例如這篇的H1只有2個，如果對第3個取值，則會回傳空值

取出所有 < H3 > 標籤下面的 < a > 
--------------------------- 

	p doc.xpath('//h3/a')

result:

	<a href="www.google.com"> google web site </a>
	<a href="www.yahoo.com"> yahoo web site </a>

如果想取中間那排文字呢，可以用 text 來取出，如   

	p doc.xpath("//h3/a").text

result:

	google web site yahoo web site

指定 item [0] 的 text  

	p doc.xpath("//h3/a")[0].text
	
result:

	google web site 

這邊會把所有的 html tag 清除掉，如果你想保留，則必須要 to_html 的方法

	p doc.xpath("//h3/a")[0].to_html
	
result:

	<a href="www.google.com"> google web site </a>
	
取出 Tag 的屬性
--------------------------- 
例如`href`的值呢，則可以用`['attribute']`來取出

	p doc.xpath("//h3/a")[0]['href']
	p doc.xpath("//h3/img")[0]['src']

result:

	www.google.com
	goodPic-1.jpg 

使用`//`來找出某種特定屬性 
--------------------------- 
當然，你也可以組合`//`來找出某種特定屬性 

	p doc.xpath( "//@href" ) # ( @代表屬性的意思)

result:

	www.google.com
	goodPic-1.jpg 

找出特定 id or class 的 tag
---------------------------
div 標籤通常會有 div id 屬性，所以要取某個特定的 div 標籤其實也很簡單  
如：找出所有 div tag 中，其屬性 class 為 `div_1` 的值出來  

	p doc.xpath("//div[@class='div_1']")

result:

	<div class="div_1">
		<h2> this is h1 in div_1 </h2>
	</div>
	
body tag 中，其屬性 id 為 "div_1" 的值出來  

	p doc.xpath("//body[@id='story_body']")

result:

	<body id="story_body">
		<h2> this is h2 in story_body </h2>
	</body>


如果還要知道詳細的用法，可以參考 [w3schools 的 xpath](http://www.w3schools.com/xpath/xpath_syntax.asp)

編碼
=============
編碼是一件很頭痛的事情，如果想要指定編碼可以藉由下列的方式
```
	url = 'http://www.google.com/search?q=sparklemotion'
	doc = Nokogiri::HTML( open( url ), nil, 'UTF-8' )
```

參考資料
================
如果覺得這邊的範例不夠，底下有幾個 link 可以參考看看

[Nokogiri 詳細使用方法](http://51itbk.sinaapp.com/14_02_2012/177.html)  
[我抓網頁資料的方法 使用 Ruby ](http://blog.ericsk.org/archives/732)  
[Ruby 編碼學問很大](http://blog.sammylin.tw/nokogiri-encoding/)  

[Get link and href text from html doc with Nokogiri & Ruby?](http://stackoverflow.com/questions/9336039/get-link-and-href-text-from-html-doc-with-nokogiri-ruby)  
[Searching an HTML / XML Document](http://nokogiri.org/tutorials/searching_a_xml_html_document.html)  
[Quick start to parsing HTML](https://github.com/sparklemotion/nokogiri/wiki)  
[Getting Started with Nokogiri](http://www.engineyard.com/blog/2010/getting-started-with-nokogiri/)  
[nokogiri tutorials](http://nokogiri.org/tutorials)  
[Parsing an HTML / XML Document](http://nokogiri.org/tutorials/parsing_an_html_xml_document.html)  
[The XML Example Document](http://www.w3schools.com/xpath/xpath_syntax.asp)  


