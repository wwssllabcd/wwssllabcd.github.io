title: "Hexo 安裝教學、心得筆記"
date: 2014-12-22 16:48:07
tags: [How-To, Hexo]
toc: true 
---

自從用了 Octopress ( static page + markdown 寫 blog 後 )，我發現我回不去了以前那些所謂聰明的 blog system，以前在看 blog 的時候就超討厭 blog 自動產生一些 tag 等東西而把網頁弄得超慢，
明明每次生成的東西都一樣，每一次都要自動產生的什麼鬼的，看了就不爽  

不過 Octopress 很久沒更新，且又只能限定使用 Ruby 1.9.3，造成很多不方便，所以我又跳槽到同樣是 Static Page 的 Blog System -- [Hexo]，
特別是作者 [tommy351] 是台灣人，所以用起來更是愉快，
這邊有[作者對 hexo 的介紹]    
![Hexo 可建構在 GitHub 上面，為什麼放這張圖是因為它可愛 XD](https://lh4.googleusercontent.com/-zW5Mdi4neqc/VJkouOcaeCI/AAAAAAAAswE/EMMODfCyNTU/s0/GitHub_icon.jpg)

安裝 Hexo 
=============

安裝 Hexo 所需檔案
-----------
Hexo 是建構在 Node.js 上面，所以第一步就是安裝 Node.js，到 http://nodejs.org/ 下載並安裝，
安裝好 Node.js 後，在程式集中會出現 `Node.js command prompt`，點開後會進入到命令提示字元，接下來輸入

	npm install hexo-cli -g

而在 Ubuntu 下面安裝則要加上 sudo， 之後不再詳述此點
	
	sudo npm install hexo -g

安裝好後可以鍵入 hexo 看看有沒有反應，若有反應就代表安裝好了

	hexo # 測試 hexo 是否被正確安裝

<!-- more -->

建置你的 Hexo Blog
-----------
選定你所要的目錄後(這邊取名叫`Blog`)，輸入

	hexo init Blog
	cd Blog
	npm install

接著再增加這些套件，這些套件會被安裝在`node_modules`中

	npm install hexo-renderer-ejs --save
	npm install hexo-renderer-marked --save
	npm install hexo-renderer-stylus --save

這樣就完成了初步的建置，這樣大致的就完成了建置了，簡單吧，若要檢視Blog可以使用

	hexo g  # 產生 blog
	hexo s  # 讓 blog 可以在 local 端檢視
	
然後打開瀏覽器，輸入 http://127.0.0.1:4000 就可以看到你的 Blog 了

把 Blog 放在 Github 上
-----------
首先先開一個 Repostory，取名叫 Blog 後，記住該 Repostory 的 clone 路徑後，打開本地端 Blog 中的_config.yml，
尋找 `deploy:`後，type 輸入 github，repository 就去 github 那邊，把你專案的 repo 路徑抄在這，最好是選 ssh 的，
branch 選 gh-pages `(固定，很重要，因為 github 固定以此 branch 作為網站的目錄)`，
最後會長成像以下這個樣子

	deploy:
		type: github
		repository: git@github.com:yourname/yourRepo.git
		branch: gh-pages

記住，repository 後面的 yourname，請改成你的帳號，而 yourRepo 就是你剛剛取的名子，其實這個也是 git clone 使用的路徑  
若真的不行的話，請參考[如何搭建一個獨立博客]，這邊就不再詳述  

PS: 而新版的 Hexo 已把 deploy 的方式改變，詳見 [Hexo Deployment](http://hexo.io/docs/deployment.html)  

若發生 ERROR Deployer not found: git，請執行以下指令試試

	npm install hexo-deployer-git --save

接下來找到 URL 的部份，root 那邊必須要跟你的 repo 一樣，如下所示

	# URL
	## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
	url: http://xxx.github.io/Blog
	root: /Blog/

設好後，回到 nodejs，鍵入

	hexo d  # 部署 blog 到 GitHub 上
	
就可以上傳你的 blog 到 github 了

使用標準的 markdown
-----------
由於新版的 Hexo 使用 hexo-renderer-marked 來控制 Markdown ，而所以還要在調整一下，
在 _config.yml 中，鍵入以下參數   

	marked:
		gfm: true
		pedantic: false
		sanitize: false
		tables: false
		breaks: false
		smartLists: false
		smartypants: false

其中 breaks 是控制斷行的，一般來說 markdown 是採用兩個空格( two space )來代表`<p>`，
若喜歡用 markdown 的格式的話，這邊要設為 false，建議除了 gfm 是設 true 之外. 都設為 false 比較好  
		
建立新文章
-----------

	hexo new "postName"  # 建立一個新的文章
	
增加其他功能
============

加入留言 
-----------
使用 [Disqus]，在_config.yml中，尋找 disqus_shortname ，並把後面加成你 Disqus 的 id 

	# Disqus
	disqus_shortname: 名稱

加入 Google Analytics
-----------
Hexo 有兩個 _config.yml，一個在根目錄，一個則是 theme 使用，這邊的就要用到 theme 的 _config.yml，
而以預設的 theme 來做範例，編輯 theme 目錄下的 _config.yml

	./themes/light/_config.yml
	
找到`google_analytics:`後，把 ID 貼在這邊即可

加入 TOC ( Table Of Content) 
-----------
以預設的 theme 為例，在.\themes\landscape\layout\_partial\article.ejs中，找到`<%- post.content %>`後，
再把以下的段落，加在`<%- post.content %>`之前

	<% if(post.toc == true){ %>
			<div id="toc" class="toc-article">
				<%- toc(post.content) %>
			</div>
	<% } %>

以上段落就是加入 TOC 的位置，適時上可以在任何你喜歡的地方加入  
然後在每篇文章的屬性那邊，加入

	toc: true 
	
就可以決定要不要打開該文章，而這邊可以利用 markdown 的`===`與`---`做 TOC 一級、二級的控制  
題外話，這邊也可以觀察到，post 其實就是對應到文章的屬性  

	post.content: 文章內容
	post.title: 文章標題
	post.toc: 這邊 toc 屬性其實是自己加的，若沒設定應該是 null 的值吧
	
加入 sitemap 
-----------
使用 sitemap 可以讓搜尋引擎可以快一點把你的網站抓回去, 加入的方法也很簡單, 鍵入
	
	npm install hexo-generator-sitemap --save
	
然後在 `_config.yml` 中加入以下選項即可

	sitemap:
		path: sitemap.xml

加入 RSS Feed 
-----------
使用 feed 可以讓別人訂閱你的文章, 鍵入
	
	npm install hexo-generator-feed

然後在 `_config.yml` 中加入
	
	#Feed Atom
	feed:
		type: atom
		path: atom.xml
		limit: 20
		
即可


從 Octopress 轉換到 Hexo
============
其實要轉換不會太麻煩，只是有幾個地方要注意的就是了  

網站的根目錄
-------------
在 _config.yml 中，網站的根目錄最好設為 `blog`(注意大小寫)，如下所示

	root: /blog/

Octopress 的文章放在 hexo 上 Compile
-------------
文章可以直接放過去，只是 categories 要換成 tag 會比較好，不換也可以，只是 hexo 會變成 categories 而已

新文章的格式
-------------
Octopress 產生的 markdown 檔案是有帶日期的，但 Hexo 沒有，如果想讓 Hexo 帶日期，可以修改 _config.yml中的 `new_post_name`，
改成下列格式即可
	
	new_post_name: :year-:month-:day-:title.md
	
不過這樣改，hexo 產生文章的 link 依然是以 title 為主，這樣剛好相容於 octopress，真是太棒了  

換佈景主題
============
可先到 [Hexo Theme] 先決定好一個主題後，把該主題clone 下來，放在/theme 目錄中，
如我想要換 daisy 這個主題，則需要做以下步驟

	git clone https://github.com/imbyron/hexo-theme-daisy.git ./themes/daisy  

接著在_config.yml中，尋找 theme 這個關鍵字後，輸入剛載下來的那個目錄的名稱，如下所示

	theme: daisy
	
修改 default theme 的封面
-----------
修改封面，各家的都不太一樣，要自己去找，而修改 landscape 的封面為以下兩個  
修改封面圖案(圖片大小為 1920x1200 )

	YourBlogPath/themes/landscape/source/css/images/banner.jpg

使用外部封面圖案

	YourBlogPath/themes/landscape/source/css/_variables.styl

裡面的`banner-url = "images/banner.jpg"`  
在這邊提供一個小技巧, 因為原始的 theme 是搭配黑色為底， 所以我們可以去 Flickr 那邊找CC授權的[銀河(milk way)]照片，看到滿意的就抓下來當封面
這樣就可以弄得跟預設的不一樣了

[Flickr_milkyway]: https://www.flickr.com/search?text=Creative+Commons+Milky+Way

修改 theme -- cover
-----------
Hexo 的 theme 中，有個叫 [theme cover] 做的蠻可愛的，也蠻乾淨的，所以這邊拿來當作範例  

* 修改封面: 在 theme/cover下的 _comfig.yml 中修改   
* 修改中間的小圖: 放在/source/logo.png  
* 修改 brower 的 ico: 放在/source/favicon.ico

而_comfig.yml中的 auto_change 最好也關掉，如果不想再文章下面出現分享到微博之類的，就把 add this 也關掉，
留言系統，若想使用原本的 default 值，就把 comment_provider 槓掉即可

Troubleshooting
===============

執行 hexo g 後，出現 warning: LF will be replaced by CRLF
---------------
輸入以下指令

	git config --global core.autocrlf false

在其他的地方(別台電腦) check out 下來你的 blog src  
---------------
因為 Hexo 的 Module 是跟著目錄的，所以如果把 code check out 下來，還是要在該目錄執行  

	npm install

或者是

	npm install hexo-renderer-ejs --save
	npm install hexo-renderer-marked --save
	npm install hexo-renderer-stylus --save

這三個，你會觀察到 ./node_modules 多了一些檔案  

出現 src refspec master does not match any
---------------
檢查一下你的 repo 是否還沒有上傳檔案，你可以先把 src 上傳之後，在做 deploy 的動作看看

出現 spawn ENOENT at errnoException childprocess.js:1001:11
---------------
* 輸入 git 檢查一下你的環境是否可以使用 git，如果不行的話，要把 git 加入到環境變數中才行   
* 輸入 node -v  檢查一下你的環境是否可以使用 node    
* 輸入 hexo --version 檢查一下你的環境是否可以使用 hexo   

出現 fatal: Could not read from remote repository.
---------------
檢查 Git ssh 設定，或者是在 Git Bash 中執行

出現 events.js:85 的錯誤
---------------
執行 hexo d 發生錯誤，錯誤訊息如下所示

	INFO  Deploying: git
	INFO  Clearing .deploy folder...
	INFO  Copying files from public folder...
	events.js:85
		throw er; // Unhandled 'error' event
            ^
	Error: spawn git ENOENT
		at exports._errnoException (util.js:746:11)
		at Process.ChildProcess._handle.onexit (child_process.js:1053:32)
		at child_process.js:1144:20
		at process._tickCallback (node.js:355:11)

解法: 在 Git Bash 中執行 hexo d 即可

執行 hexo d 出現 permission denied (publickey).
---------------
* 先檢查自己的 .ssh 目錄( windows 是放在 C:\Users\yourname\.ssh\  ) 有沒有放入 id_rsa
* 若有id_rsa檔案，但是還是有問題的話，可能是權限問題，特別是用 window 系統 copy 過去，修改成以下即可

	cd ~/.ssh
	chmod 700 id_rsa

Reference
============
[如何在 Windows 上設定 node.js 的開發環境](http://dreamerslab.com/blog/tw/how-to-setup-a-node-js-development-environment-on-windows/)  
[如何搭建一個獨立博客——簡明Github Pages與Hexo教程](http://www.jianshu.com/p/05289a4bc8b2)  
[hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)  
[升級hexo的一些坑](http://code.wileam.com/update-hexo/)  
[hexo的私人訂製](http://blog.sunnyxx.com/2014/03/07/hexo_customize/)


[銀河(milk way)]:https://www.flickr.com/search/?q=milkway
[如何搭建一個獨立博客]:http://www.jianshu.com/p/05289a4bc8b2
[Hexo Theme]:https://github.com/hexojs/hexo/wiki/Themes
[theme cover]:https://github.com/daisygao/hexo-themes-cover
[Disqus]:https://disqus.com/
[tommy351]:https://twitter.com/tommy351
[Hexo]:http://hexo.io/
[作者對 hexo 的介紹]:http://zespia.tw/blog/2012/10/11/hexo-debut/
