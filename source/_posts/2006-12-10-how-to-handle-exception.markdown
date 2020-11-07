---
layout: post
title: "例外處理"
date: 2006-12-10 17:59
comments: true
tags: [OO, C++]
---

「不會處理就別抓他...」  
「那如果有Exception呢？」我狐疑的說  
「就讓他出去阿，你不會處理就不應該抓住他!」  

這句話是在技術架構競賽前幾天，我跟文君討論的時後他跟我講的，我始終覺得不應該讓使用者看到死亡畫面，那時候還懵懵懂懂， 直到兩天前，就付出了代價了  

<!--more-->

這邊先提出兩個問題  

1. 如果 function 是被呼叫者，例如一些你所擁有的 domain Object 裡面的 function，你會覺得需要寫 Exception 嗎？  
2. 如果你 Catch 了 domain Object 裡面的 function( 例如你的 domain Object 裡面有個 GetUserById 的 Function)，他需要try catch嗎？   

也許你會情不至禁的寫 Try Catch，然後把 Exceptin 寫入 Log 裡  
```
	Public Function GetUserbyId(byval UserId as String ) as myUserObject
		try
			Dim UserObject as myUserObject
			Select * from UserTable Where UserId = '" & UserId & "' "
			.....
			do O/R Mapping
			.....
			return UserObject
		Catch ex
			WriteLog(ex)
		End try
	End Function
```

乍看之下，似乎沒什麼不妥，其實這個段程式犯了很多的錯誤，怎說呢？

這個 function 很顯然的，他應該是被 call 者，也就是還有一個 Function在stack裡，他在等著GetUserbyId回傳一個UserObject，可是當GetUserbyId發生錯誤時，他居然吃案了 ～

{% blockquote %}
吃案是不允許的...
{% endblockquote %}

也許你會問  
「在 Catch 裡面 WriteLog 不是做了處理了嗎？這也算吃案嗎？」  
「沒錯，算!」  

因為你外面的那個Caller並不知道有exception發生，GetUserbyId 這個 Function 偷偷的把錯誤給吃掉了，
警察局把案子吃掉了，警政署長每次都覺得，這個警察局表現很好，都沒發生案件，這其實是錯誤的，其實你可以寫Log，像是
```
	Catch ex
		WriteLog(ex)
		Throw
	End Try
```

也就是說你寫完 Log 後，你必須要再把他 throw 出去，讓外面的也知道，可是你在裡面寫 Log，你外面的 caller 一定也會寫 Log ，因為他在外面也是包著一層 Try Catch，就像是
```
	Public Sub Save()
		Try
			....
			....
			GetUserbyId("sa" )
			....
		Catch ex
			WriteLog(ex)
	End Try
```

這代表了一個 Exception，寫了兩次的 Log，若今天有一個 Function 被 Call 的很深，假設是十層，
若今天有十個使用者在存取你的 Web Application，你就會發生100次的IO動作，效能一定會被拖到，
好吧，那你想「那GetUserbyId就改成只丟throw就好了」，如下面所示
```
	Public Function GetUserbyId(byval UserId as String ) as myUserObject
		Try
			....
			....
		Catch ex
			Throw
		End try
	End Function
```
可是，你不覺得很可笑嗎？我把他 Catch 住，就只為了 Throw，那我不 catch 他，他也會自己 throw 阿，那我幹嘛 Catch 他阿？  
沒錯，所以說你不應該Catch他，程式就會變成這樣
```
	Public Function GetUserbyId(byval UserId as String ) as myUserObject
		....
		....
	End Function
```
「那那那....我要 Try Catch幹嘛？」  
回到最上面那句老話  

{% blockquote %}
不會處理就別抓他...
{% endblockquote %}

你應該為你的程式做一些防禦，例如說發生 Exception 時，給個預設值，好讓其他的程式繼續執行下去，也就是說「你會處理的，才需要處理」，
上面的 GetUserbyId 萬一掛掉了，很可能 Call 他的人就會拿到一個 Null，進而使接下來執行的程式當掉  

* 如果是 GetAdminMail 的話，也許妳可以在 Exception 發生時 Catch 他，然後 Return 一個預設值，也就是 Return a neutral value  
* 如果你是寫輸出畫面的程式，那麼有一個點發生了 Exception，你可以選擇 Return the same answer as the previous time  
* 如果你是寫核子反應爐的程式的話，發生 Exception，你可能要把系統給 Shutdown  

不同的程式應該是用不同的 Approach

根據 Code Complete 那本書上寫的8.3章節 Error Handling Techniques
妳可以選擇

* Return a neutral value
* Substitute the next piece of valid data
* Return the same answer as the previous time
* Substitute the closest legal value
* Log a warning message to a file
* Return an error code
* Call an error processing routine/object
* Display an error message wherever the error is encountered
* Handle the error in whatever way works best locally
* Shutdown

不過通常來講，Domain Object中，妳可以先Defensive自己
例如GetUserMailById這個function
通常來說，唯一的ID會取出唯一的東西，此時你就可以先判斷是不是唯一
若不是唯一，可以丟出exception
如Throw New NullReferenceException("e-mail address not found by UserId --->" & UserId)
代表用UserId 找不到唯一的 mail address ，妳最好順便也把傳入的參數給丟出去，好讓debug容易一點...
```
	Public Shared Function GetUserMailById(ByVal UserId As String) As String
		Dim strSql As New StringBuilder
		Dim dt As DataTable
		strSql.Append("SELECT USR_ID，USR_EMAIL FROM VW_USER WHERE USR_ID ='" & UserId & "'")
		dt = mdVcDbComm.GetDB(strSql.ToString)

		If dt.Rows.Count = 0 Then
			Throw New NullReferenceException("e-mail address not found by UserId --->" & UserId)
		End If
       Return dt.Rows(0).Item("USR_EMAIL").ToString()
    End Function
```
而關於寫Log的部分，可以寫在Globel裡面的Application error event裡面

而如果要自訂錯誤的畫面的話，可以在該page裡面抓Start_Error這個Event