---
layout: post
title: "Policy-Based Programming"
date: 2013-10-08 19:02
comments: true
tags: [Modern C++ Design, C++] 
toc: true
---

考慮以下的 code 是`找出vector"中，相同的item出來，並放在另一個vector中`，
但是有時後放入vector 的item是 native type( 如 int， char， 等)，有時卻是使用pair type，
而這兩個 template 基本上只差在一個地方，就是取 addr 的部份，
一個 native 版本是直接取，如`addr = (*iDBlock);` ，而 pair 版本是取他的 first 出來，如 `addr = (*iDBlock).first;`

Native type 版本

```
	template<bool flag, class T, class U>
	static void findDupItem(vector<T>& source, vector<U>& dupColl){
		typename vector<T>::iterator iDBlock;
		dupColl.clear();

		T addr=0;
		
		for(iDBlock = source.begin(); iDBlock!=source.end(); iDBlock++){
			addr = (*iDBlock);
			if( _isHit( addr ) == true ){
				dupColl.push_back((*iDBlock));
			}
		}
	}
```

Pair 版本

```
	template< class T, class U >
	static void findDupItem(vector< pair<T, U> >& source, vector< pair<T, U> >& dupColl){
		typename vector< pair<T, U>  >::iterator iDBlock;
		dupColl.clear();

		T addr=0;
		for(iDBlock = source.begin(); iDBlock!=source.end(); iDBlock++){
			addr = (*iDBlock).first;
			if( _isHit( addr) == true ){
				dupColl.push_back((*iDBlock));
			}
		}
	}
```
<!--more-->

基本上 99% 的東西都一樣，違反 DRY( Don't repeat yourself)原則，但那要如何把他合併起來呢？
使用一個 flag 也許可以解決問題

	if ( isPairType ){
		addr = (*iDBlock).first;
	}else{
		addr = (*iDBlock);
	}
	
這解法我們早就做過 N 遍了--   一個醜陋但堪用的解法  
你我都知道，這個解法一旦複雜度變高可能會變得難以 maintain，接下來就是看誰倒楣

也許一個 function pointer 去包裝他可能是一個好方法，如

	static void findDupItem(vector< pair<T, U> >& source, vector< pair<T, U> >& dupColl, FP pFun ){
	..略..
	for(iDBlock = source.begin(); iDBlock!=source.end(); iDBlock++){
		addr = pFun(iDBlock);
		..略..
	}

嗯，是可以解，但沒有驚喜，有沒有一個更好的解法呢？

Policy-Based Programming
===================
[Policy-Based Class Design](http://zh.wikipedia.org/zh-tw/%E5%9F%BA%E6%96%BC%E5%8E%9F%E5%89%87%E8%A8%AD%E8%A8%88)
首見於 Andrei Alexandrescu 出版的 《Modern C++ Design》一書，詳細的內容可以參考該書的第一章(見本篇後記)。     

* Interface design 的缺點  
Interface design 不是不好， Interface design 有時也會出現力有未逮的情況，
當軟體規模擴大到一定程度時，有時會很難避免出現某些 sub class 繼承 interface 時，不需要該 interface 的某些 constraints(約束條件)，
對於實務上遇到這種況狀，通常會故意忽略掉那些參數(例如傳一個 Null 值進去)，好讓 compiler 可以順利編譯，
這種語法有效，但語意無效的介面意味著 interface 出現過度設計的 bad smell。    

* 把每個功能切割成為小class  
對於 interface 出現過度設計的狀況，縮小設計規模可能是一種解法，但是這種作法又會產生大量的設計組合，
以 smart point 為例，你就會有一堆 class 如下   


	SingleThreadSmartPtr  
	MultiThreadSmartPtr  
	RefCountSmartPtr   
	RefLinkSmartPrt  
	

若增加一個選項則會面臨大量設計組合，複雜度曲線上升  

* 利用多重繼承來處理設計組合？  
若是利用繼承 BaseSmartPoint去組合SingleThread，RefCount 這些 class 所產生的 SmartPoint，
的確是可以讓設計組合降低，但除非設計單純，否則大多數得小心繼承帶來的痛苦，
特別是要去協調那些 class的運轉，就實務上來說繼承並不討喜。  

繼承組合而來的 class 面對著型別又有著困擾，假設你使用一個 DeepCopy Class 來為你的 SmartPointer 實作 DeepCopy，
但是 DeepCopy 是怎樣的介面呢？ 舉例來說，假設他要回傳某個東西，那回傳的 type 又是什麼？   

多重繼承本質上的確是一種`組合`，但似乎沒有辦法單獨的解決這種問題，特別是 user 在設計時，面對型別的多樣性。   

* Template 帶來曙光  
哪裡型別最多？就是 Template，那裡擁有大量的型別，而這兩種設計並不衝突，有時還相輔相成，
比較一下多重繼承與 template，例如 多重繼承 往往缺乏型別，而 temaplte 擁有大量型別。   

而且一個良好的設計應該在編譯時期強制表現出大部分的 constraints (約束條件)，
而 Template 剛好可以在 compile time 表現出 constraints 的機制。   

用多重繼承 + template 來實現有機會產生非常彈性的裝置來當作我們的設計元素。
  

實作 Policy Classes Design
===================
根據上述的解釋，修改 code 如下  

先把 GetItem使用 Policy class 封裝起來，
由於我們不知道他的 ReturnType 與 Collection 的 Iterator 是什麼，反正缺 Type 就往 Template 那邊找就對了

```
	template< typename Iter, class RT >
	struct GetNornalData{
		static RT getItem(Iter iter){
			return (*iter);
		}
	};
	
	template< typename Iter, class RT >
	struct GetPair_first{
		static RT getItem(Iter iter){
			return (*iter).first;
		}
	};
```

以 vector<int> 來說，這邊的 ReturnType 就是 Int， 而 iter 就是 Vector<int>::iterator 了，
有了policy，接下來就可以改寫 main function 了

```
	template< class T, class ReturnType, class GetDataPolicy >
	class FindDpu : public GetDataPolicy
	{
	public:
		static void run(vector<T>& source, vector<T>& duplicateColl){
			typedef vector<T>::iterator Iter;

			Iter iter;
			ReturnType addr=0;

			...略...

			for(iter = source.begin(); iter!=source.end(); iter++){
				addr = getItem(iter); // 利用繼承而獲得的 function
				...略...
			}
		}
	};
```

使用的時候如下，先宣告你要那個 Policy，再利用 Policy 去產生哪種 FindDpu class
```
	vector<int> source, target;
	typedef GetNornalData< vector<int>::iterator, int > Policy;
	
	FindDpu< int, int, Policy >  fvpd_int;
	fvpd_int.run(source, target);
```
如果是 pair 的時候，可以使用
```
	typedef pair<char, char> MyPairType;
	
	vector<MyPairType> source, target;
	typedef GetPair_first< vector<MyPairType>::iterator, MyPairType::first_type > Policy;
	
	FindDpu< MyPairType, char, Policy > fvpd_pair;
	fvpd_pair.run(source, target);
```

一點也不美麗的呼叫方式
===================
的確是使用了 Policy，但使用上前置作業要好幾行，
在使用之前先宣告想要的 Policy，呼叫的時候傳入Policy ，這似乎合情合理，
但使用 pair 的時候傳入 pair policy，使用int的時候傳入 int Policy 又顯得多餘又危險 -- 我總不可能使用 vector < pair > 傳入 int Policy 吧？


模版偏特化( partial specialization )
===================
能不能看到我傳 pair 的時候就預設 pair Policy 呢？翻了一下書，利用模版偏特化( partial specialization )也許有機會

可以看以下例子
```
	template<typename T>
	Class Test;

	template<>
	Class Test<char>{
		typedef char ReturnType;
		string show(){
			pringf("ReturnType = char");
		}
	}
	
	template<>
	Class Test<int>{
		typedef int ReturnType;
		string show(){
			pringf("ReturnType = int");
		}
	}
```
此時如果使用

	Test<char> t;
	t.show()
	
則會顯示

	`ReturnType = char`
	
compiler 藉由型別推導選擇了不同的 class，看來我們有機會藉由輸入不同的 Type，而去選擇不同的 Class 

重新打造 Policy Class
===================
根據我們傳入的 T，利用 compiler 型別推導機制，來幫 Policy 作偏特化，藉以自動選擇哪種 Policy，
重新設計 Policy
```
	template<typename RT > 
	struct GetDataPolicy;

	template< typename RT> 
	struct GetDataPolicy < std::vector< RT > >	
	{
		...略...
	};

	template< typename RT, typename U > 
	struct GetDataPolicy < std::vector< std::pair<RT, U> > >
	{
		...略...
	};
```
這樣一來，只有 `T = pair<RT, U>` 的時候，compiler 會自動推導 pair Policy 為最佳解，其餘的都是 vector< RT > 解，
而且是符合最佳解的才會被編譯出來，也就是說，沒用到的 compiler 根本就不會編譯他  

回到主程式來，先利用預設模版，然後拿 T 當作 Policy 的參數，而 compiler 會拿 T 去 Policy 作型別推導找出最佳解出來  

	template< class T, class ReturnType, class GetDataPolicy<T> >
	class FindDuplicateItem : public Policy
	{
		...略...
	}

拿掉多餘的 template parameter -- 使用 Traits
===================
觀察 main function 中的 ReturnType

	template< class T, class ReturnType, class GetDataPolicy<T> >
	class FindDuplicateItem : public Policy
	{
		...略...
	}
	
基本上 ReturnType 也是一開始就知道的資訊，若使用偏特化的機制，也是讓我們有機會拿掉他，
檢視一下整個程式可能會需要用到的 Type  

* getItem 所用到的 ReturnType
* 用以作for 迴圈的 vector<T>::iterator


只要在 Policy 中重新定義就好

	template< typename RT> 
	struct GetDataPolicy < std::vector< RT > >	
	{
		typedef typename RT ReturnType;
		typedef typename std::vector< RT >::iterator CollItor;
		...略...
	};

甚至可以建立一個專門作 Trais 的 template，這樣一來所需要的資訊都在 Traits 裡面了
```
	template< typename Return_Type, typename Colls_Iter  > 
	struct GetDataPolicyTraits
	{
		typedef typename Return_Type  ReturnType;
		typedef typename Colls_Iter   CollsIter;
	};

	template< typename RT> 
	struct GetDataPolicy < std::vector< RT > >	
	{
		typedef GetDataPolicyTraits<RT, typename std::vector<RT>::iterator> Traits;
		typename Traits::ReturnType getItem(typename Traits::CollsIter iter){
			return (*iter);
		}
	};
```

在 main function 中，使用的時候像這樣
```
	template< typename T, typename Policy = GetDataPolicy<T> >
	class FindDuplicateItem : public Policy
	{
	public:
		void run(T& source, T& duplicateColl){
			Policy::Traits::CollsIter  iter;
			Policy::Traits::ReturnType addr=0;
			...略...
		}
	}
```

有趣的是，明明 main function 傳入的是一個被泛化的 Type T，
居然可以取出`std::vector< RT >` 或是`std::vector< std::pair<RT, U> >`的 ReturnType 與 Iterator

Traits 是一種`把T丟進去某個特徵萃取機制中，取出特定特徵`的技巧，
當然不僅僅只是定義 type 而已，參考以下程式
```
	template <typename T>
	class TypeTraits
	{
	private:
		template <class U> struct PointerTraits
		{
			enum { result = false };
			typedef NullType PointeeType;
		};
		template <class U> struct PointerTraits<U*>
		{
			enum { result = true };
			typedef U PointeeType;
		};
	public:
		enum { isPointer = PointerTraits<T>::result };
		typedef PointerTraits<T>::PointeeType PointeeType;
		...
	};
```

現在我們可以來猜看看，vector<int>::iterator 是不是int 的 pointer ?
```
	int main()
	{
		const bool iterIsPtr = TypeTraits< vector<int>::iterator >::isPointer;
		cout << "vector<int>::iterator is " << iterIsPtr ? "fast" : "smart" << '\n';
	}
```


把一切組裝起來
===================
藉由 Traits 把缺的 main function 中的 Type 給補足
```
	template< typename Return_Type, typename Colls_Iter  > 
	struct GetDataPolicyTraits
	{
		typedef typename Return_Type  ReturnType;
		typedef typename Colls_Iter   CollsIter;
	};
	
	template<typename RT > 
	struct GetDataPolicy;

	template< typename RT> 
	struct GetDataPolicy < std::vector< RT > >	
	{
		typedef GetDataPolicyTraits<RT, typename std::vector<RT>::iterator> Traits;
		typename Traits::ReturnType getItem(typename Traits::CollsIter iter){
			return (*iter);
		}
	};

	template< typename RT, typename U > 
	struct GetDataPolicy < std::vector< std::pair<RT, U> > >
	{
		typedef GetDataPolicyTraits< RT, typename std::vector< std::pair<RT, U> >::iterator > Traits;
		typename Traits::ReturnType getItem(typename Traits::CollsIter iter){
			return (*iter).first;
		}
	};

	template< typename T, typename Policy = GetDataPolicy<T> >
	class FindDuplicateItem : public Policy
	{
	public:
		void run(T& source, T& duplicateColl){
			Policy::Traits::CollsIter  iter;
			Policy::Traits::ReturnType addr=0;

			duplicateColl.clear();

			//let sorting time approach to Big(1) by bitMap
			const size_t MAP_SIZE = 0x80000;
			BYTE map[MAP_SIZE]={0};

			for(iter = source.begin(); iter!=source.end(); iter++){
				addr = this->getItem(iter);
				if( _isHit( addr, map, MAP_SIZE) == true ){
					duplicateColl.push_back((*iter));
				}
			}
		}
	private:
		bool _isHit(ULONG value, BYTE* pMap, size_t mapSize){

			if( value >=mapSize){
				throw MyException(UTI_PARAM_ERROR, "value>=mapSize"); 
			}

			bool result = true;
			if( pMap[ value ] ==0){
				pMap[ value ] =1;
				result = false;
			}
			return result;
		}
	};
```

呼叫的時候，也只要傳入最低限度的type即可

	vector<int> source, target;
	FindDuplicateItem< vector<int> > fvpd;
	fvpd.run(source, target);

這邊沒有選擇使用`template template parameter`的方式去作，是因為我希望可以在源頭就直接抽換掉 Policy Class 而不受到 < T > 的影響。

後記
===============
其實對於軟體工程而言，完成一件事情可以用很多種不同的作法，而且解法似乎一樣好，但是也許就是選擇太多，
導致沒經驗的工程師選擇了對未來有 side effect 的方案，導致專案越來越難進行，
這只是一個簡單的 template ，幹嘛搞的那麼複雜，主要的還是觀念上的改變 -- 使用 if else 與使用 Policy 的區別， 

以下兩個 link 對於 Policy-Based Class Design 我覺得寫的很好，版主很用心再寫，可以參考看看  
* [探索Policy-Based Class Design新視界](http://blog.monkeypotion.net/gameprog/beginner/exploring-the-field-of-policy-based-class-design)  
* [深入Policy-Based Class Design新大陸](http://blog.monkeypotion.net/gameprog/advanced/diving-into-policy-based-class-design)  

本篇很多案例是參考 [Modern C++ Design](http://jjhou.boolan.com/jjtbooks-modern-cpp-design.htm)，
及 [C++ template 全覽](http://shopping.pchome.com.tw/?mod=item&func=exhibit&IT_NO=DJAR0F-A16046100)，

而 Policy-based programming 在 Modern C++ Design 的 ch1 有詳細的介紹，
棒的是，侯傑有 [ch1~ch4 的預覽版本可供下載](http://jjhou.boolan.com/jjtbooks-modern-cpp-design.htm)(真的是佛心來著)，有興趣的朋友可以參考看看




