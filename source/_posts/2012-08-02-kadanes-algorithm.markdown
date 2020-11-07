---
layout: post
title: "Kadane`s algorithm"
date: 2012-08-02 22:19
comments: true
tags: [Algorithm, C++, Programming pearls]
---
 
此篇文章有些程式碼遺失，使用時請注意!  
Kadane's algorithm 是 Programming Pearls CH8 有提到  
我只是純粹有興趣，畢竟他可以把 O(n*3) 變成 O(n)  
題目不長，請見最下面  
我後來有找一下，題目可以跟這個比擬，找出哪天買股票與賣股票的時間點是賺最多的  
[聽說是 Google Phone Interview 的考題](http://www.mitbbs.com/article_t/JobHunting/31731345.html)

<!--more-->

教主提出來的 k-th largest sum   

	Output: the kth largest sum(a+b) possible. where
	a (any element from array 1)  
	b (any element from array 2)  

這問題小弟駑鈍，只算出    

	n*k + (n-1)*k +..+ 1*k  
	= k(n +(n-1) +(n-2) + ... 1)
	= k*n(1+n)/2
	= O(n*k)

Orz  

剛睡到一半突然全想通了  
如果 K-th 的題目真的是兩個 array  a[n],b[k] 中，找出任意兩數最大值的組合的話  
那還真的是 O(n+k)  
因為 a+b 的最大數必定為 a 陣列中的最大數，與 b 陣列中的最大數的相加  
找出 a[n] 的最大數只需要 O(n)  
找出 b[k] 的最大數只需要 O(k)  
答案就是 O(n+k)  

這種問題還是不要睡前想比較好，以免睡不著  
如果我猜的沒錯的話，google 的那個問題是個爛問題  
然後 programming pearls 的問題，也是 O(n) 就可以解了  

Programming pearl 的範例
==================================
有一個 array x[10] = {31,-41,59,26,-53,58,97,-93,-23,84}  
找出一區間，令 x[i+...+j] 其值是最大的  
例如隨便找兩組  
第一組: i=0, j=2，其總和為 31+(-41)+59    = 49  
第二組: i=0, j=3，其總和為 31+(-41)+59+26 = 75  

所以第二組比第一組來的好，以此類推  
故簡易的演算法為 O(n*3)  

	int const n=10;
	char x[n] = {31,-41,59,26,-53,58,97,-93,-23,84}
	int maxValue = 0;
	int i, j, k;
	for(i=0; i<n; i++){
		for(j=i; j<n; j++){
			int sum=0;
			for(k=i; k<j; k++)({
				sum+=x[k];
			}
			maxValue = max(maxValue, sum);
		}
	}
	

O(n*2)的演算法(書上教的)
==================================
sum其實可以重複利用  

	char x[n] = {31,-41,59,26,-53,58,97,-93,-23,84}
	int const n=10;
	int maxValue=0;
	int sum=0;
	int i, j;
	for(i=0; i<n; i++){
		sum=0;
		for(j=i; j<n; j++){
			sum+=x[j];
		}
		maxValue = max(maxValue, sum);
	}

O(n) 的演算法 
==================
Programming pearls 的解法我是這樣思考  

1. 如果該陣列全部都是正值，那答案就非常簡單，就是全部 array 的總和  

2. 假設從 x[0] 開始往右累加，如果累加的值"不為負"，答案就跟 1 一樣  
   就是 array 如果是 {31,-1,59}，因為31-1是30，還是正數，所以代表加上去之後，結果會跟1推導的一樣  
 
3. 上面兩點討論了累加值都是正的狀態，這邊討論累加值出現負的狀態，也就是說假設從 x[0] 開始往右累加，出現累加的值"為負"的話  
   
這邊特別討論為負的情況，以題目來講
假設 1：該 x[i] 陣列，i=0 開始累加到 x[1] 的時候，發現其總和 sum_0_1 < 0 （即 31-41 = -10 )
假設 2：sum x[2~9] 的總和為 sum_2_9(先不管他是否為正為負)  

根據以上假設，這也意味著 sum_0_1 + sum_2_9 絕對不可能比 sum_2_9 本身還要大 
簡單的來說，有一數字 X (不論是正數，負數)，加上一個負的值，絕對不會比 X 本身來得大，所以出現負值的區間可以直接丟棄
 
也就是說，一旦發生負值，答案就要一定會存在於 sum x[2~9] 這個區間的最大值  

程式如下  

	int endingHere=0;
	int startHere=0;
	int maxSum=0;
	int curSum=0;
	int const n=10;
	char x[n] = {31,-41,59,26,-53,58,97,-93,-23,84};
	
	for(int i=0; i<n; i++){
		curSum+=x[i];
		if(curSum >= maxSum){
			//record maxSum & endingHere
			maxSum = curSum;
			endingHere = i;
		}
		if( curSum < 0 ){
			//reset current sum, just like point to x[6~10]
			curSum=0;
			maxSum=0;
			
			//reset startHere, endHere
			startHere = i+1;
			endingHere = i+1;
		}
	}


所以這個的 time-complexity 也就從 O(n*3) 變成 O(n)  
演算法也變成 liner time algorithm 了  

