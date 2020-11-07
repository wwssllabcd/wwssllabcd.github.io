---
layout: post
title: "從舊 code 過渡到新 Code 的心得"
date: 2007-03-10 19:25
comments: true
tags: [C++]
---

血淚心得阿  
在新與舊的 code 中，因為舊的 code 太爛，所以你會想要重新建立新的 code，若想要從旁邊建立起新的 code 時，這樣必然會出現兩種 function 做同一件事情，但是為了保險起見，你不會用你的 code 替換掉舊的 code  

但是就長期來說，你的計畫一定是逐步以新的 code 的替換掉舊的東西，在還沒替換掉舊的 code 之前，你一定會 maintain 兩個功能類似的 code(一個是新的，一個是舊的)，
如果是你一個人再寫 code，你一定不會忘記，改了舊的 code 後，也要把新的 code 改過，但是往往維護的人是不只你一個的，所以有些人根本不知道有新的 code 這檔事  

到最後就會出現舊的 code 雖然被修改，但是新的 code 沒有被改到，而當過了一個很長的時間後，你對你自己新的 code 已經有信心了，此時你在著手替換掉舊的 code 時會遇到幾個問題，卻發現，以前解過的 bug 又復活了，原因是舊的 code 更新了，新的 code 並沒有，這時候災難隨之而來  

<!--more-->

要解決這個問題  
你可以在你新與舊的 code 之間寫上 Assert 碼 如下 (註：f 代表新的 code，m_flash 是舊的 code)  



	#ifdef _DEBUG
	Flash f;
	try{
		f = f.getFlash(1,m_Drive);
	}catch (exception ex){
		CString what(ex.what());
		Add_Report(4,what);
		return(ERR_NORMAL_STOP);
	}
	ASSERT( f.getBlockType() == m_Flash.FlagLB);
	ASSERT( f.getLcType() == m_Flash.FlagMLC);
	ASSERT( f.pagePerBlock == m_Flash.PagePerBlock);
	ASSERT( f.secPerBlock == m_Flash.SecPerBlock);
	ASSERT( f.getTotalBlock() == m_Flash.TotalBlock);
	ASSERT( f.blockPerChip == m_Flash.BlockPerChip);
	ASSERT( f.hasCopyBack == Utility::toBool(m_Flash.FlagCopyBack) );
	ASSERT( f.hasCacheProgram == Utility::toBool(m_Flash.FlagCacheProgram) );
	ASSERT( f.pagePerBlock == m_Flash.PagePerBlock);
	#endif

這樣可以確保新的 code 與舊的 code 是否真的一致，也減少出錯的機率  
還有，面對有賠償問題的時候  
寫 code 的策略應該是，能出錯的就讓他出錯，最差最差狀況就是你的程式不停的跳出 Alert 警告  

不過這總比以免沒卡到，讓客戶把產品做了出去，那時候在 call back 就賠不完了  

