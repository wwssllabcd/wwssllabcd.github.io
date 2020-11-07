---
layout: post
title: "Serialize 你的物件，使用 boost Serialize"
date: 2006-12-07 09:57
comments: true
tags: [How-To, Boost, C++]
---

有時候我們必須要把某些東西的資料保存下來，在 VC 中，雖然 CObject 類別提供了 Serialize 的功能，但是也只限定於繼承 CObject，而且使用方法不太直覺，
而有時候我們會使用 STL 容器來裝載某些類別，特別是自訂的類別，VC++ 雖然提供了許多的 Serialize 的功能，但是卻不能支援 STL 容器的 Serialize 的功能。   

本篇介紹如何 Serialize 一個 "裝有任何型態的 STL 容器"，在這邊物件的 Serialize 功能是使用 boost 的 Serialize 庫，
你可以對於你的容器做 Serialize 的動作，並且可以很方便的用 text , XML 方式做 Serialize 的儲存媒介。   
<!--more-->

下載並安裝 Boost
---------------
這裡就不多講了，請參照 [Boost 的安裝方式](http://wwssllabcd.github.com/blog/2012/08/03/how-to-complie-boost-lib/)，我使用的是 boost 1.5.0

使用 Boost 的 Serialize 庫 Serialize vector 容器
------------------
在使用之前，你一定要把 VC 的 RTTI(property page->C/C++->Language->Enable Run-Time Type Info) 一定要選上 (Yes)，不然會 compile 失敗。  

接下來就是使用 Serialize 的功能嚕，在使用的時候你必須 include 一些檔案。  

	#include <boost/serialization/vector.hpp>
	#include <boost/archive/xml_oarchive.hpp>
	#include <boost/archive/xml_iarchive.hpp>
	#pragma comment(lib, "libboost_serialization-vc71-mt-s-1_50.lib")

`vector.hpp`： Serialize vector 的 hpp，如果要使用其他容器請 include 其他的 hpp 檔
`xml_oarchive.hpp`： Serialize Output 成 xml 需要用的到 hpp  
`xml_iarchive`：Serialize 讀進來會用到的  
`pragma comment(lib, "libboost_serialization-vc71-mt-s-1_50.lib")`：我們目前使用的是 vc71 版本，多執行序的靜態類別庫  

這個選項會依照你 compiler 的選項而改變，請注意是否跟你的 compiler 選項相同，接下來就是要 Serialize 成 XML File，我們只要指定路徑就可以了，例如 xx.xml or c:\xx.xml 就可以了，如果你要 Serialize 一個 int 的 Vector，你可以寫成這樣  

	void save(string filePath, const vector<int>& coll)
	{
		std::ofstream ofs( filePath.c_str() );
		ofs.close();	
		boost::archive::xml_oarchive oa(  );
		oa << BOOST_SERIALIZATION_NVP( coll );
	}
	
VC LINK 2019 Error
------------------
在這邊要注意的是，全部有關 Template 的 code 都必須寫在.h 裡面，因為會有所謂的 template 的 LNK 2019 錯誤，在 [這個網站](http://blog.donews.com/forient/category/5824.aspx) 上有提到，以下引用該網站的解法

`主要的原因是模板類在編譯以前需要做一次類型參數與原有代碼的整合過程，在上例中要把 int 型參數和模板類的定義生成一份基於 int 的類定義，然後編譯。
在鏈接的時候鏈接器在.h 文件中找不到 TPL 函數關於 int 類型的定義，所以報錯 LNK2019。`

兩種解決方法：  

1.  在要被調用的函數定義名前加關鍵字 export。比如：  
	`export typ GetSecondMember() const;`  
	可惜該關鍵字不被大多數 CPP 編譯器支持 (包括 VS.net7.1)，forget it!
2.  把 typ.cpp 裡的代碼放在 typ.h 中，刪除 typ.cpp 文件。然後編譯即可。這樣子，編譯器可以找到基於 int 類型類定義，通過！:-)

所以這邊採用第二種解法，也就是把 .cpp 的 code 放到.h 中，這樣一來無論是什麼樣的 vector 都可以被接受了

valarray 的問題 (boost 1.50)
------------------
出現現象
	C:\Program Files\Microsoft Visual Studio .NET 2003\Vc7\include\valarray(314): error C2059: 語法錯誤 : ')'
	C:\Program Files\Microsoft Visual Studio .NET 2003\Vc7\include\valarray(321): error C2143: 語法錯誤 : 遺漏 ')' (在 '}' 之前)
	C:\Program Files\Microsoft Visual Studio .NET 2003\Vc7\include\valarray(321): error C2238: 在 ';' 之前有未預期的語彙基元

解決方法: 只要在  

1. `#include WinDef.h`之前，`#define NOMINMAX`
2. 或是在專案設定的 Preprocessor 裡，輸入`NOMINMAX`

就可以關閉 max, min macro 了，關鍵字:`boost/serialization/vector.hpp c2334 NOMINMAX`  

	
Serialize 自訂的 Object
------------------
到目前為止，你應該可以完成 Native 類的 Serialize，但是這往往不夠，如果你要在 vector 裡面放你自己的 class 的話，你就必須在你自己的 Class 中提供 Serialize 的功能了，假設你有個 class 叫 MyObject

	class MyObject
	{
		string value;
		int startByte;
	};
	
由於 boost 不認識你的 class，但他把 serialize 的責任交給 class 的設計者，你必須為你自己的 class 提供 Serialize 的功能，而 boost 規定這要是一個模版函數   
如：`template void serialize(Archive & ar, const unsigned int version)`

	class MyObject{
		public:
		MyObject(void);
		~MyObject(void);
		string value;
		int startByte;

		template
		void serialize(Archive & ar, const unsigned int version)
		{
			ar & BOOST_SERIALIZATION_NVP(s);
			ar & BOOST_SERIALIZATION_NVP(startByte);
		}
	};
	
而 serialize 的 code 如下
	void save(string filePath, const vector<MyObject>& coll)
	{
		std::ofstream ofs( filePath.c_str() );
		ofs.close();	
		boost::archive::xml_oarchive oa(  );
		oa << BOOST_SERIALIZATION_NVP( coll );
	}

這樣就搞定了  

建立一個 Vector < T >  的通用 Serialize function 
----------------------------
乍看之下，vector< int > 與 vector< MyObject > 的 save function 看裡來似乎很像，同樣的 code，只是 Type 不同，
這讓我們想起了 Template，我們不應該為每個 Class 做一種 Serialize 的功能，現在應該是建立起一個模版，而且 Serialize 應該被收斂在一個地方，不應該發散，
是戴起另一頂帽子的時候了，開始 Refactory  

我們建立起一個 Serialize 的 Class，把 Serialize 的功能都丟在裡面
並且為 Vector 建立起樣版函數，首先先建立起一個叫 SerializeService 的 Class  

1. 由於我們只注重 save 與 load 的動作，所以這個 class 只有兩個 function，用 Serialize 檔案的路徑當作參數 (表示我們要存到哪)  
2. 建立起 vector 的樣版函數，讓放入 vector 的各類別都可以 Serialize   

sample code 如下：
	
	#pragma once
	#include <boost/serialization/vector.hpp>
	#include <boost/archive/xml_oarchive.hpp>
	#include <boost/archive/xml_iarchive.hpp>
	#include <iostream>
	#pragma comment(lib, "libboost_serialization-vc71-mt-s-1_50.lib")
	using namespace std;
	class SerializeService
	{
	public:
		SerializeService(void){};
		~SerializeService(void){};

		template<class T>
		void save(string filePath, const vector<T>& coll)
		{
			std::ofstream ofs(filePath.c_str());
			ofs.close();
			boost::archive::xml_oarchive oa(ofs);
			oa << BOOST_SERIALIZATION_NVP(coll);
		}

		template<class T>
		load(string filePath, vector<T>& coll)
		{
			ifstream ifs(filePath.c_str());
			if(!ifs){
				throw Exception("can`t find the file By Path in SerializeService::load() ");
			}

			try{
				boost::archive::xml_iarchive ia(ifs);
				ia >> BOOST_SERIALIZATION_NVP(collnew); 
			}catch(exception e){
				string msg,err(e.what());
				msg += "Parse XML file of Serial Number error";
				msg += "\n\rPlease check the format of XML file or rebuild another one";
				msg += "\n\rOrigion error is :";
				msg += err;
				throw Exception( msg.c_str() );
			}
		}

	};

	
這邊是提供 service 的 SerializeService，那對象物體呢？通常來說直接在 MyObject 裡面寫 serialize function 即可，不過 MyObject 就必須跟 boost 綁在一起，有點不划算。其實這是可以脫耦的，利用繼承方式，建立一個 MyObjectForSS 就可以了

	class MyObjectForSS : public MyObject
	{
	public:
		MyObjectForSS(void){};
		~MyObjectForSS(void){};
		template<class Archive>
		void serialize(Archive & ar, const unsigned int version)
		{
			ar & BOOST_SERIALIZATION_NVP( value );
			ar & BOOST_SERIALIZATION_NVP( startByte );
		}
	};

這個作法的好處是，其他 proejct 沒有用到 boost 的 code ，是不需要 include boost 的，而新增一個 sub class 並對他加入 boost 的功能並不會對你新的 code 造成什麼影響，除非你開始使用他


	
參考網站:  
http://blog.csdn.net/conyzhou/archive/2005/02/04/279871.aspx  
http://blog.csdn.net/mslk/archive/2005/11/08/525278.aspx  
http://blog.csdn.net/ShowLong/archive/2006/04/19/668923.aspx  
http://www.stlchina.org/twiki/bin/view.pl/Main/BoostChina  
http://blog.donews.com/forient/category/5824.aspx  