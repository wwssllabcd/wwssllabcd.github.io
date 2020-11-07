---
layout: post
title: "VirtualBox 出現 virtualbox vt-x is not available 錯誤訊息"
date: 2012-08-03 17:56
comments: true
tags: [Virtual Box]
---

<!--more-->
直接講解法吧，到  
	user/your account/library/virtualbox/machines/ 你的專案名稱
找出 xx.xml  
使用筆記本打開，尋找   
	<hardwareVirtEx enabled="true"/>
改成 `false` 即可  