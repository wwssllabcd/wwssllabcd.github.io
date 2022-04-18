---
layout: post
title: "Serialize 你的物件，使用 boost Serialize"
date: 2006-12-07 09:57
comments: true
tags: [How-To, Boost, C++]
---

有時候我們必須要把某些東西的資料保存下來，在 VC 中，雖然 CObject 類別提供了 Serialize 的功能，但是也只限定於繼承 CObject，而且使用方法不太直覺，而有時候我們會使用 STL 容器來裝載某些類別，特別是自訂的類別，VC++ 雖然提供了許多的 Serialize 的功能，但是卻不能支援 STL 容器的 Serialize 的功能。  

本篇介紹如何 Serialize 一個 "裝有任何型態的 STL 容器"，在這邊物件的 Serialize 功能是使用 boost 的 Serialize 庫，你可以對於你的容器做 Serialize 的動作，並且可以很方便的用 text , XML 方式做 Serialize 的儲存媒介。   

## 下載並安裝 Boost

這裡就不多講了，請參照 [Boost 的安裝方式](http://wwssllabcd.github.com/blog/2012/08/03/how-to-complie-boost-lib/)，我使用的是 boost 1.79

## 使用 Boost 的 Serialize 庫 Serialize vector 容器

接下來就是使用 Serialize 的功能嚕，在使用的時候你必須 include 一些檔案。  

```c
#include <boost/serialization/vector.hpp>
#include <boost/archive/xml_oarchive.hpp>
#include <boost/archive/xml_iarchive.hpp>
#pragma comment(lib, "libboost_serialization-vc143-mt-sgd-x32-1_79.lib")
```

`vector.hpp`： Serialize vector 的 hpp，如果要使用其他容器請 include 其他的 hpp 檔
`xml_oarchive.hpp`：output xml 需要用的到 hpp  
`xml_iarchive.hpp`：input xml 需要用的到 hpp    
`pragma comment(lib, "libboost_serialization-vc143-mt-sgd-x32-1_79.lib")`：我們目前使用的是 vc143 版本，這個會看你當下使用的狀態而 include 不同的 serialize 類別庫  

接下來就是要 Serialize 成 XML File，我們只要指定路徑就可以了，例如 xx.xml or c:\xx.xml 就可以了，如果你要 Serialize 一個 int 的 Vector，你可以寫成這樣  

```c++
void save(string filePath, const vector<int>& coll)
{
	std::ofstream ofs(filePath.c_str());
	assert(ofs.good());
	boost::archive::xml_oarchive oa(ofs);
	oa << BOOST_SERIALIZATION_NVP(s);
	ofs.close();	
}
```

## Serialize 自訂的 Object

到目前為止，你應該可以完成 Native 類的 Serialize，但是這往往不夠，如果你要在 vector 裡面放你自己的 class 的話，你就必須在你自己的 Class 中提供 Serialize 的功能了，假設你有個 class 叫 MyObject

```c++
class MyObject
{
	string value;
	int startByte;
};
```
	
由於 boost 不認識你的 class，但他把 serialize 的責任交給 class 的設計者，所以你必須為你自己的 class 提供 Serialize 的功能，而 boost 規定這要是一個模版函數，如下所示

```c++
class MyObject{
public:
	MyObject(void);
	~MyObject(void);
	
	string value;
	int startByte;

	template<class Archive>
	void serialize(Archive & ar, const unsigned int version)
	{
		ar & BOOST_SERIALIZATION_NVP(value);
		ar & BOOST_SERIALIZATION_NVP(startByte);
	}
};
```

	
而執行 serialize 的 code 如下
```c++
void serialize_to_xml(estring filePath, const vector<T>& s)
{
	std::ofstream ofs(filePath.c_str());
	assert(ofs.good());
	boost::archive::xml_oarchive oa(ofs);
	oa << BOOST_SERIALIZATION_NVP(s);
	ofs.close();	
}

void some_function(){
	MyObject obj;
	
	vector<MyObject> colls;
	colls.push_back(obj);
	
	serialize_to_xml("your/path/filename.xml", colls);
}

```

這樣就搞定了  


## 建立一個 vector< T > 的通用 serialize function 

乍看之下，vector< int > 與 vector< MyObject > 的 save function 看裡來似乎很像，同樣的 code，只是 Type 不同，這讓我們想起了 template，我們不應該為每個 Class 做一種 serialize 的功能，現在應該是建立起一個模版，而且 serialize 應該被收斂在一個地方，不應該發散，是戴起另一頂帽子的時候了，開始 refactor

我們建立起一個 serialize 的 Class，把 serialize 的功能都丟在裡面，並且為 vector 建立起樣版函數，首先先建立起一個叫 SerializeService 的 Class  

1. 由於我們只注重 save 與 load 的動作，所以這個 class 只有兩個 function，用 serialize 檔案的路徑當作參數 (表示我們要存到哪)  
2. 建立起 vector 的樣版函數，讓放入 vector 的各類別都可以 serialize   

Sample code 如下：

```c++
#pragma once
#include <boost/serialization/vector.hpp>
#include <boost/archive/xml_oarchive.hpp>
#include <boost/archive/xml_iarchive.hpp>
#include <iostream>

#pragma comment(lib, "libboost_serialization-vc143-mt-sgd-x32-1_79.lib")

using namespace std;

class SerializeService
{
public:
	SerializeService(void){};
	~SerializeService(void){};

	template<class T>
	void save(string filePath, const vector<T>& colls)
	{
		std::ofstream ofs(filePath.c_str());
		assert(ofs.good());
		boost::archive::xml_oarchive oa(ofs);
		oa << BOOST_SERIALIZATION_NVP(colls);
		ofs.close();	
	}

	template<class T>
	void load(string filePath, vector<T>& colls)
	{
		ifstream ifs(filePath.c_str());
		assert(ifs.good());
		boost::archive::xml_iarchive ia(ifs);
		ia >> BOOST_SERIALIZATION_NVP(colls); 
		ifs.close();
	}

};
```

這樣一來就能建立一個通用 vector 的 serialize service 了

## 使用 text 的方式做 serialize

若想要換成 text 的方式的話，只要把 xml_iarchive 與 xml_oarchive 換成 text_oarchive 與 text_iarchive，並且 include 相關 header 就可以了
```c++
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
```

## 參考網站

http://blog.csdn.net/conyzhou/archive/2005/02/04/279871.aspx  
http://blog.csdn.net/mslk/archive/2005/11/08/525278.aspx  
http://blog.csdn.net/ShowLong/archive/2006/04/19/668923.aspx  
http://www.stlchina.org/twiki/bin/view.pl/Main/BoostChina  
http://blog.donews.com/forient/category/5824.aspx
