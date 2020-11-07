---
layout: post
title: "在Heroku上，安裝 Wordpress"
date: 2013-01-08 17:18
comments: true
tags: [How-To, Heroku, Wordpress]
---
其實在 Heroku 上安裝 Wordpress 不會很難，不過閱讀之前，你可能先要知道 Heroku 與 git 的基本操作，建議可以先參考以下網站  
[用 Heroku 架設 Wordpress 網站](http://yapeshu.invenio.us/2011/10/web-heroku-Wordpress.html)

Heroku 端設定
---------------------
* 使用 Command line，鍵入 Heroku create 建立一個 app
*  上去 Heroku 網站改名
*  使用 git 下載該 app
*  去 Heroku 網站中的 add-on ， 選擇 Heroku PostgreSQL(free)，並且把他加到該 Wordpress 中  

使用 Clinet 指令為   
	heroku addons:add heroku-postgresql:dev
	
*  使用 Command line， 打 Heroku config ，看看有沒DB的資訊出來，如果有東西出來(填 wp-config.php 要使用到) 就可以進行下一階段

<!--more-->

下載 Wordpress 與 PostgreSQL for WordPress
----------------------

*  [下載 Wordpress](https://Wordpress.org/download/)
*  由於 Wordpress 是使用 mysql，而 Heroku 只有 PostgreSQL ，所以必須要裝一個轉換外掛 [PostgreSQL for Wordpress (PG4WP)](https://Wordpress.org/extend/plugins/postgresql-for-Wordpress/)

*  解開 Wordpress，並且丟到使用 git下載回來的那個空 repo 中
*  解開 PostgreSQL for WordPress，把裡面的 `pg4wp` 丟到 `\wp-content` 中 (可以看看readme.txt)
*  從 `pg4wp` 資料夾中，copy `db.php` 到`\wp-content`
*  到 Wordpress 根目錄，找到`wp-config-sample.php`後，複製一份並更名為 `wp-config.php`
*  修改 `wp-config.php`，根據 Heroku config 所給的資訊，填入 db 參數

填法如下(若不懂可以看我最下面的參考網站連結)
	postgres://DB_USER:DB_PASSWORD@DB_HOST/DB_NAME

上傳至 Heroku Web
--------------------
使用 git指令為   
	git add .  
	git commit -m "first init"  
	git push heroku master  
	
上傳完就完成囉！接下來就是去  
	http://你的APP名稱.herokuapp.com/wp-admin/install.php 
去確認 WP 有安裝完成就可以了，若失敗，可以看看 heroku logs


Troubleshooting 
-------------------------
無法建立 database 連結
由於 Heroku Postgres 9.1 does not allow connection to 'template1'   

請去`pg4wp` 資料夾中的`driver_pgsql.php`找到以下這段
	// While installing， we test the connection to 'template1' (as we don't know the effective dbname yet)
	if( defined('WP_INSTALLING') && WP_INSTALLING)
		return wpsql_select_db( 'template1');
	return 1;

修改成
	// While installing， we test the connection to 'template1' (as we don't know the effective dbname yet)
	if( defined('WP_INSTALLING') && WP_INSTALLING)
		return wpsql_select_db(DB_NAME); // Heroku Postgres 9.1 does not allow connection to 'template1'          
	return 1;

參考網站(Reference)
---------------------------	
[用 Heroku 架設 Wordpress 網站](http://yapeshu.invenio.us/2011/10/web-heroku-Wordpress.html)
[[WEB] 安裝 GitHub 上的 WP 模版於 Heroku](http://yapeshu.invenio.us/2011/10/web-github-wp-heroku.html)