---
layout: post
title: "神秘的 Resource.close() & 自動資源管理"
date: 2012-08-03 19:15
comments: true
tags: [C++]
---

很久很久以前，我曾經有這樣的疑問，對於某一種 resource (例如 dbconn,Device 等等)，明明他就可以自動管理的很好，那為
何還會有.close() 的 Funciton，出現在他的 class 中呢？  

<!--more-->

舉個例來說，有某個 Resource 使用方式如下  
	{
		Resourcer
		r.open()
		...
		...
		r.close()
	}

好啦，就算沒有 close 他，他照樣會把 Resource 給釋放的好好的，也就是說跟以下的 code 跟上面的 code 是一樣的效果  
	{
		Resourcer
		r.open()
		...
		...
	}

哎呀，叫我選要用哪一個 function，我應該會選第二種，看來第二種應該比第一種受歡迎，所以之後常常看到大家都使用第二
種的寫法，可以不用寫 close，事實上，不寫 close 也不會有問題  

問題來了，很久很久以前，當我看到第二種的寫法的時候，心理就有一個念頭"為何會有一個 r.close() 這個 function"，他到
底要什麼時候使用呢？因為就算沒這個 function，他也活的好好的，那設計這個 API 的人，為何會設計出這一個 funciton 呢？  

產生這個疑問為止，到了今天，我才了解為何會有這個東西存在，看到這裡為止，如果你是往 memoryleak 的方向思考，我只能
說你猜對了一半嚕，可是我讚嘆的是另一半的理由  

怎說？  

一個 Resource 使用 destructor 去釋放資源是一件很常見的手法，也許.close() 是在解構式出現了 memoryleak，所做出的補救手
法，但是他的另一種理由才是另我覺得驚訝的，但是我們先撇開 destructor 中 memleak 的問題，在這裡 Mayer 提供了另一種思考方式  

當你沒有 close 的時候，你只能依賴著 destructor，可是有了 close()，你還是可以依賴 destructor 幫你完成釋放的工作，但
是 close() 提供了一個讓你自行處理的機會，如果你不作，destructor 依然會幫你好好的完成這分工作  

撇開 code 來說，居然會想到要開放一個機會給使用者處理，真是太厲害了  
若你不想處理也沒關係，只是代表你放棄機會，destructor 依然會幫你弄的好好的  
也就是說，有.close() 比 autoreleaseresource 的思維更加細密，所以這就是為何會有一個看似累贅，卻又暗藏神秘道理的 function 了  
神奇吧!  

有時在使用別人的 (特別是自家人的) API 時，老是覺得很難用很不順手，也許就是這種小地方沒有替使用者考慮進去喔  

