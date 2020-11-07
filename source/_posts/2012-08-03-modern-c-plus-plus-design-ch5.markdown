---
layout: post
title: "Modern C++ Design CH5"
date: 2012-08-03 19:36
comments: true
tags: [Modern C++ Design, C++, OO]
---

我 chapter 5 的讀書心得，理解之後趕快把他記下來，以免太久又要重新思考
而 chapter 5.1 ~ 5.3 屬基本功，請自行參閱原書籍

<!--more-->

chapter 5.4
-----------
我們要做一個 Loki Functor 物件 必須要可以跟一般 funciton 一樣可以下參數，還有回傳值，也就是要做到以下程式碼的功能
	Functor cmd( max() );
	int test = cmd(3,5); 

首先先來做一個 Functor Class 吧
	class Functor
	{
		public:
			operator()();
	};

通常一個 function 會有以下兩點特徵

1. 像一般 function 擁有回傳值
2. 下參數的功能

所以我們上述建立的 class 這樣寫可能是不夠的，畢竟我們是要模擬 Function，這兩個要求很合理

處理回傳型態問題
-----------
先來解決回傳參數的問題，似乎不難做到，利用 template 來讓 Return Type 的型別泛化就完工了
所以 Functor 就變成了 Template 了。  

	template<typename ResultType>
	class Functor
	{
		public:
			ResultType operator()();
	};

這問題真好解決，不是嗎？

處理傳入參數的問題
-----------
接下來要模擬一般 funciton 一樣可以傳入參數的問題
因為參數的型態未知，所以想當然爾，要用 template 來處理

本來很直覺的應該是以下的寫法

	template<typename ResultType>
	class Functor
	{
		public:
			ResultType operator()();
	};
	template<typename ResultType, typename param1>
	class Functor
	{
		public:
			ResultType operator()(param1);
	};

但是很遺憾的，Compiler 不會讓你那麼爽，因為Compiler 不會允許你使用同名而參數不同的 template，
但是要把 Functor 都分別改名為 

	class Functor_NoParam
	class Functor_1
	class Functor_2

是很蠢的事，所以又要拐彎了，在 Chapter3 我們有學到一個"保存一堆 type"的工具，就是 Type list，這時候就派上用場了，
以下的code意味著，特定長度的 parameter，接下來，作者想要把參數的問題抽象化，就做了一個 FunctorImpl 的多型介面
從此 Functor 這層不用再去思考一共幾個參數的問題了

	template <typename R>
	class FunctorImpl<R, NullType>
	{
		......
	};
	
	template <typename R, typename P1>
	class FunctorImpl< R, TYPELIST_1(P1) >
	{
		......
	};
	
	template <typename R, typename P1, typename P2>
	class FunctorImpl< R, TYPELIST_2(P1, P2) >
	{
		......
	};

而再使用 Functor 包裝這個 FunctorImpl，所以 Functor 裡面會有一個 FunctorImpl 的 data member(這邊採用的是 handle/body 的手法)
也就是定義一個 FunctorImpl 的 point，再把他包在 auto ptr 裡

	template <class TLIST>
	class Functor
	{
		public:
			....
		private:
			typedef FunctorImpl Impl;
			std::auto_ptr<Impl> spImpl_;
	}

所以說，要建立一個 Functor 物件的話，上面的例子就要改成
	Functor<int, TYPELIST_2<int, int> > cmd;
	int test = cmd(3,5);

CH 5.5
------------
由於 FunctorImpl 才是真正會做事的物件，Functor 只是包在外面
如果要做事的話，Functor 當然是轉 call FunctorImpl 嚕，其實呼叫很簡單，就像下面一樣，以下為 1 個參數的呼叫

	class Functor
	{
		R operator()(Parm1 p1)
		{
			return (*spImpl_)(p1);
		}
	}
但是這邊有個困擾點，我怎麼知道 Parm1 所代表的型別呢？，所以為了要解決這個問題，
首先先來設計各種不同長度傳入參數，這邊試著把每個參數取出，CH3 開發的 TypeAtNonStrict 擁有取出 type 的能力

	template <typename R, class TList>
	class Functor
	{
		typedef TList ParmList
		typedef typename TypeAtNonStrict<TList, 0, EmptyType>::Result Prarm1
		typedef typename TypeAtNonStrict<TList, 1, EmptyType>::Result Prarm2
		… 略 …
	}

這邊試著從 TList 取出傳入參數的 type，並且重新定義叫 Prarm1、Prarm2，也就是說，parmN 是TList中的第 N 個 type，
有了 Parm 之後，就可以傳入參數並呼叫 function 了

	template <typename R, class TList>
	class Functor
	{
		typedef TList ParmList
		typedef typename TypeAtNonStrict<TList, 0, EmptyType>::Result Prarm1
		typedef typename TypeAtNonStrict<TList, 1, EmptyType>::Result Prarm2
		… 略 …
		
		public:
			R operator()()
			{
				return (*spImpl_)();
			}
			R operator()(Parm1 p1)
			{
				return (*spImpl_)(p1);
			}
			R operator()(Parm1 p1, Parm2 p2)
			{
				return (*spImpl_)(p1, p2);
			}
			… 略 …
	};

也就是說，當你在呼叫

	Functor getMaxCmd(&max());
	int test = getMaxCmd(3,5); 

其實你是在呼叫
	(*spImpl_)(p1, p2);

5.6 Handling Functors 
----------------------
好啦，介面都 ok 了，但是，要如何把 function 放入到 functor 呢？就像這樣
	Functor getMaxCmd( max() );
	
我們是把 max() 放進去了，但是以上的討論根本就沒把這塊接起來
首先無庸置疑的一定要有一個建構式

	class Functor
	{
		public:
			Functor(const ？);
	};

這邊的問號似乎要填仿函式的 type，那麼仿函式的 type 是什麼呢？要知道這個問題的答案之前，得先瞭解仿函式的定義
根據作者給仿函式的定義
"Functors are loosely defined as instances of classes that define operator()"
也就是鴨子類型 (duck typing)，簡單的說，不管什麼鬼東西，只要裡面有 operator() 就可以稱的上是"仿函式"
也就是說 Class A、B、.....etc 任何 class 都有機會成為仿函式

那麼 Functor(const ？); 這個問號的答案也是呼之欲出了。答對了，就是我們的好朋友`template`

	template <class TList>
	class Functor
	{
		… as above …
		public:
			template 
			Functor(const Fun& fun);
	};


定義了介面，那 implement 呢？
作者又把"由外部傳進來的 function pointer"，找到了一個管理者，叫做 FunctorHandler
FunctorHandler 負責管理  "由外面來幫忙的客人"

也就是說，作者希望 Functor 不再為外部 function 而操煩了...
講技術一點，就是把所有不關 functor 的東西封裝在 FunctorHandler 裡
讓 FunctorHandler 去照顧這位"由外部遠道而來的貴客"

也就是說，作者希望做到 Functor 保有對外部 function 的 Smart  Pointer 的建構方式可以做到像這樣
	spImpl_(new FunctorHandler (fun));

functor 擁有一個 spImpl_ 的指標，指向由建構式傳進來的外部 function 的參考
而這個外部 function 的存放的細節都交給 FunctorHandler 去管，functor 只管 call 就是...

複習一下我們原本的建構式

	template <class TList>
	class Functor
	{
		… as above …
		public:
		template 
		Functor(const Fun& fun);
	};


現在變成

	template <class TList>    //跟之前的沒變
	template                  //原本是建構子的 template 被提到上面來，稱之為 member template 定義式
	Functor::Functor(const Fun& fun)
	: spImpl_(new FunctorHandler(fun)); //smart ptr 保存了 FunctorHandler
	{
	}


題外話
寫到這邊我有種感覺，其實這種架構很像是  
	Functor ----------------------> 老闆
	FunctorHandler、FunctorImpl --> 部門經理

外部的 Funciton 就是員工了，所以我對他在 FunctorHandler 裡面命名為 ParentFunctor 覺得怪怪的，因為 ParentFunctor 會給我有繼承關係的感覺。

5.7 Build One, Get One Free (做一個送一個)
---------------------------------------------
在進入主題之前先回顧一下 Loki Functor 要做到的功能
若要當成 Command Pattern 的設計工具，他最好能很方便的放入 function

我們先自己幻想一下，他應該有什麼功能  

1. 可以放入仿函式，例如	`Functor cmd(SomeFunctorObject);`
2. 當然要支援現有 function，例如 `Functor cmd(&max());`

注意：支援 function 又分 `一般 function` 及 `member Function`，像是 C style 就像是一般 function 而藉由物件呼叫的 function 就稱 `member Function`
這兩種 function 有點不一樣。  

藉由以上的討論，已經做到了仿函式的建構了，那 function 的建構方法要怎麼 implement 呢？
答案是，不用作了，compiler 送給你了

其實我們可以仔細的看看 FunctorHandler

	class FunctorHandler: public FunctorImpl
	{
		public:
			FunctorHandler(const Fun& fun) : fun_(fun) {}
			.... 略................
			ResultType operator()(typename ParentFunctor::Parm1 p1)
			{
				return fun_(p1);
			}
			.... 略................
		private:
			Fun fun_;
	};

今天你把 Funciton 傳進去，會變成怎樣呢？
假設我有一個 Function，他的程式如下

	void TestFunction(int i, double d)
	{
		.... 略.....
	}

我如果要使用他，就是把他丟到 Functor 當作建構式的參數
例如
	Functor cmd(&TestFunction);

到了 FunctorHandler 會變成怎樣？
首先一定會先進去 FunctorHandler 的建構式裡.....

	class FunctorHandler: public FunctorImpl
	{
		public:
			FunctorHandler(const Fun& fun) : fun_(fun) {}
		private:
			Fun fun_;
	};

其實這邊有個問題，就是若是傳一般函式的話，`Fun` 與 `fun_` 到底是什麼？
fun_ 應該比較好猜，他應該是一個 ptr，指向 TestFunction  (畢竟我們是傳 &TestFunction 進來)，
那 Fun 呢？如果有看過 C++ primer 的人應該就知道，
Fun 就是"函數的類型"，函數的類型只由它的返回值和参數表决定，
也就是說 `void TestFunction(int i, double d)` 他可以寫成

	void (*pTestFunction)(int,double);
	pTestFunction = &TestFunction;

也就是說，建立起一個 cmd 時，就必須先設定好他的 return Type 及參數
例

	Functor getMaxCmd(&max());
	int test = getMaxCmd(3,5);

代表這個 getMaxCmd 接受兩個 int 類的參數，執行後回傳一個 int
建立起來，用法可以這樣用

	Functor otherCmd;

代表這個 otherCmd 接受兩個參數 (一個 int, 一個 bool)，執行後回傳一個 string

	Functor otherCmd;
	string testStringCmd  = otherCmd(3,true);


待續...