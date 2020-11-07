---
layout: post
title: "Type Class 的寫法"
date: 2012-08-03 18:22
comments: true
tags: [C++, OO]
---

自從一年半前聽過文君講過一遍觀念，然後又看到 Effective C++ 後就一直很想把他寫出來的，我這邊稱為 Type Class，因為我不知道要叫這種東西是什麼，所以拿了這個詞來代替，希望有人可以糾正我這個可能是錯誤的 term 嚕  

一個程式裡，或多或少都需要用一些 type，舉個例來說，如果要對人類性別有"男"還有"女"  

<!--more-->

簡單的來說，如果有一個 Person 的 Object 的話，可能會有 se x 的屬性，用來描述人類性別  
	Person a_ban;// 產生阿扁
	a_ban.sex = ?

問號的值到底要填什麼呢？通常來說，這邊要填 man(男), 或是 woman(女)  
此時我們給一個簡單的定義，把 0 定義成男性，把 1 定義程女性  

所以 a_ban 這個人的 sex 屬性有了答案  
	a_ban.sex = 0;

有點 sense 的人，不會讓 magic number 出現，他們會這樣作...  
	#define man 0
	#define woman 1
	
或是	
	int const man(0);
	int const woman (1);

所以以上的寫法會被改成  
	a_ban.sex = man;

如果以後要加入小孩，或是老人的種類，還可以這樣擴充  
	#define man 0
	#define woman 1
	#define child 2
	#define oldMan 3
	
所以讓阿扁變成老人的話，可以這樣表示  
	a_ban.sex = oldMan ;

擴充很方便，一切的一切也是非常美好，乍看之下似乎沒有任何破綻  
而且這種 type 通常為方便起見，都會拿數值來作  
可是往往這種作法並不是很好，原因是  

1. 當有個 funciton 要傳入 humanKind 參數時，往往會因為某些因數傳錯，造成程式錯誤  
某個 function 的原型如下
	someRortine(int sex);

你卻把 age 傳進去
	someRortine(a_ban.age);

2. 有時也可以拿 aPersion.sex 來當作計算的參數，如
	aPersion.sex +=1;// 性別可以拿來加減嗎？

傳錯了不打緊，更糟的是，他還可以正常運作，直到一個月後，你的客戶打電話來找你...  
想想看這是多可怕的事情  

以上兩個案例不要覺得這不可能發生，因為他本來就是整數，這種運算本來就是合法  
你最好的盟友 compile 絕對會讓他過  

你回憶起你作 PGR 的這段時光，各種光怪陸離的事情不是沒有見過  
你總是會看到一些不可思議的程式，或是邊看說"這樣用真是亂七八糟"  
罵歸罵，可是你還是無法讓其他的 prg，不做出這種事情  
而更多的例子可以看 Effective C++，在這不在贅述.....  

這個問題的解法就是用 Type 去防禦  

那用 Type 有幾點我的心得就是  

1. 利用 funciton 回傳 Type Object
為什麼？假若你現在作一個月份的 Type  
如本想打 month(1)，變成 month(2) 等等的錯誤 (3 就在 2 的旁邊，要犯這種錯誤不是很罕見)  
何不用 Month::Jan()，或是 Month::feb()，所以你會用 Funciton 代替手打  
如  

	Month::Jan(){
		Month(1);
	}

2. 把 constructor 變成 private
既然你已經決定要用 Funciton return Object 了，可見你不能讓使用者自己產生 Object ，所有的 Object 都必需要由 Function 產生，這樣一來也可以避免手打錯的問題  

3. 有一個 operator==()  

在這之前，我蠻討厭寫 operator 的 function，不過 Type Class 的 operator 反而很好寫  
可以使用 access 到 rhs. 的 Private Property  
如  
	this->_value == rhs._value

所以整個 operator==funciton 就是  
	bool operator==(const Month& rhs){
		return this->_value == rhs._value;
	}
夠簡單吧，此外，觀察第一點  
	return Month(1);
會產生兩個 object，當你在程式中大量使用這種方法，會產生一堆這種臨時物件...  
所以最好是用 static 的來作，如在 class Month 中，放入 private static Month jan(1);  

在做 function 時，就 return 該 static object，如  
	static Month jan(1);
		Month::Jan(){
		return &jan;
	}

這裡有一個雛型，為何說是雛型，因為他還有一些進步的空間  
例如剛講的每個 function return static Object instand to return by temp Object(效能改善)  
或是建立一個 TypeClass Base 的 Policy Object，以代替每次都是一成不變得 Null Object , int _value , operator==()  等這些 function，不過已經是一個可以 Run 的 code 了  

這裡的 Type 是以 Nand Flash 的 Block Type 來舉例

程式碼 
==========================================
	class BlockType
	{
		public:
		~BlockType(void);

		static BlockType null(void){
			return BlockType(0);
		}
		static BlockType smallBlock(void){
			return BlockType(1);
		}
		static BlockType largeBlock(void){
			return BlockType(2);
		}
		bool operator ==(const BlockType& rhs) const{
			return (this->_value == rhs._value);
		}

		CString toString(void){
			CString msg;
			switch(_value){
			case 0:
				msg = “Null Block Type”;
				break;
			case 1:
				msg = “Small Block”;
				break;
			case 2:
				msg = “Large Block”;
				break;
			default:
				msg = “UnKnow Block Type”;
				break;
			}
			return msg;
		}

		private:
		BlockType(int value)
		:_value(value)
		{
		}
		int _value;
	}

而使用的時候，在某 Class 內的初始化方式為  
	MpOption::MpOption(void)
	:clockType(ClockType::null() )
	{
	}

剩下的懶得寫了...