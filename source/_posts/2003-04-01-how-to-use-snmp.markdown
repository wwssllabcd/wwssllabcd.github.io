---
layout: post
title: "SNMP 簡介"
date: 2003-04-01 08:44
comments: true
tags: [How-To, SNMP]
toc: true
---

SNMP 簡介
=============

簡單網路管理協定，可以用來監控網路裝置，簡單的說明大致上可以分為  

    MIB:存放裝置狀態的資料庫
    SNMP Agent:幫你管裝置的代理程式

而資料是存放在管理系統中，因為管理資料的複雜性，它包含多個元件：

    管理資訊結構（Structure of Management Information，SMI）
    管理資訊庫（Management Information Base， MIB）
    協定（如SNMP）

<!--more-->

SMI指定結構和描述管理資料，MIB詳細記錄各個物件，協定負責使用者和管理者之間的溝通，
SMI使用樹狀的概念，一些物件代表樹葉，也只有葉節點才有存放MIB值。
代表的物件是使用ISO ASN.1 (Abstract Syntax Notation-1)的概念。
SMI對每一個物件分配一個整數，稱為物件識別碼（Object Identifier， OID）來記錄它們在樹狀圖中的位置，
詳細說明可以參考以下連結[SNMP網路管理程序，文魁資訊股份有限公司](http://www.microsoft.com/taiwan/technet/book/tcpip/default_part2.htm)

注意:
要注意的是有些系統，有些MIB編號是浮動式的，每次 Enable 或 Disable 一個介面，該編號都會有異動。所以當MRTG
執行後，會產生 mrtg.ok 檔案，此檔案會紀錄目前哪一個編號對應到哪一個介面，當發生錯誤時，必須要自行核對此檔案，然後手動去修改介面編號的正確值。


MIB與Device
=============
每個提供SNMP的裝置都有他的MIB，裡面存放的就是此裝置目前狀態的值，例如說一張網路卡的MIB可能有下列這幾樣，
網路卡名稱，網路卡現在狀態(on，off)，網路流量...等(有點像物件導向中的物件)，而我們藉由查詢這些MIB的值來瞭解現在裝置的狀況

ANS.1與BER
=============
[ANS.1重點摘要](http://216.239.53.100/search?q=cache:dju_2EF7du8C:www.cyut.edu.tw/~icchang/course/88-1-NM/ASN1/3B4/asn1.htm+ASN.1%E4%B9%8B%E9%87%8D%E9%BB%9E%E6%91%98%E8%A6%81&hl=zh-TW&ie=UTF-8)

系統監控與MRTG
=============
如何找出監控項目，當然嚕，就是要看MIB與SMI嚕，很累的，至於SMI與Private的資料，請回到上面的SMI與MIB，有相關連結，
你也可以看看 Windows 的 MIB，在%System%/System32/下面副檔名為MIB的，用記事本打開就可以看了

MRTG 安裝
=============
這裡提供了一個[在Windows下裝 MRTG的方法](http://www.freecoolpages.com/roadhouse/awho/mrtg/win2k/mrtgwin2k.html)，寫的不錯，請自行參照

Linux的人請自行使用搜尋引擎，有很多可以讓你看，不提供連結了，你可以在 cfgmaker 下一些參數讓你的 cfg 檔產生一些變化，範例如下

	cfgmaker --global "WorkDir: c:\www\mrtg" --global "Refresh: 600" --global
	"Interval: 5" --global "WriteExpires: Yes" --global "Language: big5" --global

	"options[_]: bits" --ifdesc=descr --ifref=descr
	public@127.0.0.1

不過通常來說，我都是這樣下的(在windows下的環境)

	perl cfgmaker public@127.0.0.1 --global "WorkDir: c:\www\mrtg" --global
	"Language: big5" --output mrtg.cfg


他只是幫你做cfg檔，你也可以直接跳過，自己寫

1. 再來輸入 perl mrtg mrtg.cfg 按下 ENTER"(在這步驟的時候，你的cfg檔裡面最好不要有 RunAsDaemon:yes的指令)
2. 接著修改mrtg.cfg ，使用文書編輯軟體，在mrtg.cfg的最後加上一行 RunAsDaemon: yes之後存檔
3. 再來回到 C:\ 下輸入 start /Dc:\mrtg\bin wperl mrtg --logging=eventlog mrtg.cfg MRTG即開始工作

**Note: 關於第一點, 在這裡 Complier 的時候，若在 < table > 那邊出現錯誤，
則代表你把 HTML 斷行了，導致他們變成了數個字串，所以 MRTG 不能 Complier，
這種情況多發生在您去直接 copy 網頁上的 cfg 檔，而導致產生斷行，
有時候不能 Complier 是你的 target 有問題，有空格或是怎樣的，自行排除**

當你重開機後，他就不會幫你自動的偵測，若要再啟動，只要再做一次第三步驟就可以了，因為如果要做 Polling 的話，
就必須要加 RunAsDaemon:yes 與 Start 的指令，但是如果在第一步就先加了 RunAsDaemon:yes，則好像會過不了 complier，所以才要那磨麻煩

這邊節錄了Polling的方法，瞭解一下原理嚕，翻譯至 MRTG 的 Win32

	MRTG Installation Guide 
	MAKE MRTG RUN ALL THE TIME

	Starting mrtg by hand every time you want to run it is not going to make you
	happy I guess.

	There is a special option you can set in the mrtg configuration file so so
	that mrtg will not terminate after it was started. Instead it will wait for 5
	minutes and then run again.
	
		Add the option
		RunAsDaemon: yes
	
	to your mrtg.cfg file and start it with:
	start /Dc:\mrtg-2.9.27\bin wperl mrtg --logging=eventlog mrtg.cfg

	If you use wperl instead of perl， no
	console window will show. MRTG is now running in the background. If it runs into
	problems it will tell you so over the EventLog. 
	
想要停止 MRTG 的話，就打開工作管理員，把 wperl.exe process 停掉，假如MRTG有任何訊息要告訴你，你可以在event log這邊找到這些訊息，
假如你放一個捷徑

	Target:    wperl mrtg --logging=eventlog mrtg.cfg
	Start in:  c:\mrtg-2.9.27\bin

進去你的啟動資料夾裡， mrtg will now start whever you login to your NT box.  
若你想啟動MRTG而不需登入.[參考這個網站](http://www.firedaemon.com/mrtg-howto.html)，這裡有非常多的免費軟體可以讓你啟動 Server，包括 MRTG 的使用者  

詳細的OIDs
---------------------------
你可以利用詳細的OID 去查詢語法如下

	syntax 'OID_1&OID_2:community@router

	The following example will retrieve error counts for input and output on
	interface 1.

	MRTG needs to graph two variables， so you need to specify two OID's such as
	temperature and humidity or error input and error output.

	Example:
		Target[ezwf]: 1.3.6.1.2.1.2.2.1.14.1&1.3.6.1.2.1.2.2.1.20.1:public@myrouter

MIB 變數
------------------------
你也可以利用MRTG所定義的名稱去做查詢，你可以看看MRTG裡的mibhelp.txt 那裡列出了一些他們定義的參數，舉個例來說，我若要查ifInErrors與ifOutErrors. 就可以打入下列這樣

	Example:
		Target[ezwf]: ifInErrors.1&ifOutErrors.1:public@myrouter

他會自動去參照mibhelp.txt而找出正確的OID

LoadMIBs
---------------
Load the MIB file(s) specified and make its OIDs available as symbolic names. For better efficiancy， a cache of MIBs is maintained in the WorkDir.

	Example:(用逗號隔開)
	LoadMIBs: /dept/net/mibs/netapp.mib，/usr/local/lib/ft100m.mib

XSize and YSize  
---------------
MRTG圖形大小的預設值為 100 by 400 pixels wide (plus some more for the labels. In the example we get almost square graphs ...

Note: XSize 必須在 20 與 600之間 ; YSize 必須大於 20
	
	Example:
		XSize[ezwf]: 300
		YSize[ezwf]: 300
		XZoom and YZoom

	If you want your graphs to have larger pixels， you can ``Zoom'' them.

	Example:
		XZoom[ezwf]: 2.0
		YZoom[ezwf]: 2.0

XScale and YScale
--------------------
If you want your graphs to be actually scaled use XScale and YScale. (Beware: while this works，
the results look ugly (to be frank) so if someone wants to fix this: patches are welcome.

	Example:
		XScale[ezwf]: 1.5
		YScale[ezwf]: 1.5


note : 若想看個更清楚，可以把Xsize調成100，然後把Xscale變成4，這樣一來還是400，不會破壞美觀

YLegend， ShortLegend， Legend[1234]
-------------------------
The following keywords allow you to override the text displayed for the various
legends of the graph and in the HTML document:

YLegend 就是你圖表上Y軸所顯示的文字，但是不要取的太長的名子，否則會被忽略

ShortLegend 測量的單位(預設為'b/s') 出現在統計網頁的Max， Average and Current的單位

Legend[1234IO] 線的說明:在統計圖表最下面那邊，最多可以6個(1234io)

	Example:
		YLegend[ezwf]: Bits per Second
		ShortLegend[ezwf]: b/s
		Legend1[ezwf]: Incoming Traffic in Bits per Second
		Legend2[ezwf]: Outgoing Traffic in Bits per Second
		Legend3[ezwf]: Maximal 5 Minute Incoming Traffic
		Legend4[ezwf]: Maximal 5 Minute Outgoing Traffic
		LegendI[ezwf]: &nbsp;In:
		LegendO[ezwf]: &nbsp;Out:

Note，if LegendI or LegendO are set to an empty string with LegendO[ezwf]: The corresponding line below the graph will not be printed at all.  
以下還有一些參數，是參考網路上的資料而來的而來的，不過我有補充一些所以跟原文不同，要看原文可以去網路上搜尋

MRTG 快速安裝指導1-4
=============
這邊補充一點資料 [MRTG 快速安裝指導1-4](http://netlab.kh.edu.tw/document/%E5%BC%B5%E6%AF%93%E9%BA%9F/MRTG%20%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%9D%E6%8C%87%E5%B0%8E.htm)

Note:
在這裡Complier的時候，若在<table>那邊出現錯誤，則代表你把HTML斷行了，導致他們變成了數個字串，所以MRTG不能Complier，這種情況多發生在您去直接copy網頁上的cfg檔，而導致產生斷行

Directory[se2cc]: ROOM
指示 MRTG 將 se2cc 這一個統計項目的資料， 以及這些資料所產生的 HTML 文件與圖表， 放在 WorkDir 之下的 ROOM 子目錄中， 也就是 /home/www/flux/ROOM

這個目錄.
對MRTG參數還想知道更多嗎？全文可參考[MRTG的網站](http://people.ee.ethz.ch/~oetiker/webtools/mrtg/reference.html)

偵測CPU Loading，Process，Memory， HD(Hard Disk)Free Space

使用Windows XP SP1，並且安裝過了SNMP4PC網站的purch， MRTG是使用MRTG2-9-27， Perl使用ActivePerl-5.6.1.635-MSWin32-x86.msi， 以下是我的 cfg 檔


	CPU使用率
	YLegend[cpuPercentUse]: % Utilization
	WithPeak[cpuPercentUse]: ymw
	XSize[cpuPercentUse]: 300
	Options[cpuPercentUse]: growright，gauge
	Target[cpuPercentUse]:.1.3.6.1.4.1.311.1.1.3.1.1.2.1.5.1.48&.1.3.6.1.4.1.311.1.1.3.1.1.2.1.5.1.48:public@192.168.0.1
	MaxBytes[cpuPercentUse]: 100
	Title[cpuPercentUse]:CPUPercentUse
	ShortLegend[cpuPercentUse]: %
	Legend1[cpuPercentUse]: Proc Load in next minute
	Legend2[cpuPercentUse]: Proc Load in next minute
	Legend3[cpuPercentUse]: Maximal 5 Minute Proc Load
	Legend4[cpuPercentUse]: Maximal 5 Minute Proc Load
	LegendI[cpuPercentUse]: &nbsp;CPU使用率:
	LegendO[cpuPercentUse]: &nbsp;CPU使用率:
	PageTop[cpuPercentUse]: <H1>cpuPercentUse</H1><TABLE><TR><TD>System: </TD><TD>你的系統</TD></TR><TR><TD>HostIP</TD><TD>你的IP</TD></TR><TR><TD>Load MIB:</TD><TD>cpuPercentPrivilegedTime</TD></TR></TABLE>
	WorkDir: c:/www/mrtg
	Language: big5
	#RunAsDaemon: yes

有時候不能Complier是你的target有問題，有空格或是怎樣的，自行排除，讓我蠻驚訝的就是Target居然還支援數學運算，先乘除後加減，也可以使用括號，
其實還可以偵測 IIS 之類的，自行去SMI的Private裡的331(Microsoft)裡面找吧，包你偵測個三天三夜都還搞不完，其他的自行去研究偵測吧.......


一小部分的MIB說明
=============
注意:
要注意的是有些系統，有些MIB編號是浮動式的，每次 Enable 或 Disable 一個介面，該編號都會有異動。所以當MRTG
執行後，會產生 mrtg.ok 檔案，此檔案會紀錄目前哪一個編號對應到哪一個介面，當發生錯誤時，必須要自行核對此檔案，然後手動去修改介面編號的正確值。

SNMP漏洞
=============
由於SNMPv1在做的時候並沒有考慮到安全部分，所以有很多安全上的問題，SNMPv2，SNMPv3雖加強安全上的議題

但是基本上很多廠商還是根據SNMPv1來做，所以我們在用的SNMP就有很多安全上的漏洞
例如: BTT Software SNMP Trap Watcher 遠端阻斷式攻擊

當遠端使用者創設一個含有多於 306 個字串的 string trap 並送給監督工作站時，監督工作站的軟體將立即出現 "此程式已執行一個違規操作"的錯誤視窗。藉 Linux 的以下指令，
此攻擊方法可以一直被複製讓遠端 Trap Watcher 當機：

	$ snmptrap vulnerable.host community '' '' 0 0 ''
	interfaces.iftable.ifentry.ifindex.1 `perl -e 'print "A"x2000'`

以下連結為卓蘭實驗室的GSN-CERT/CC 第 0084 號通報 : SNMP
[相關弱點](http://cc.cles.mlc.edu.tw/article.php?sid=40)

後記
==============
我發現我這個人，只要興趣一來了，就會一頭栽進去，不眠不休的想去瞭解他，之前是JAVA，現在是SNMP，
去年聽了蔡老師的課，那時候根本不瞭解什麼是SNMP，有一次看到文儀在用What`sUP，藉由他安裝MRTG給我看，
我才有一點瞭解，而一直到安裝了第一次的MRTG，才比較清楚他整個運作的過程(果然還是要自己做一次阿)  


裝了MRTG我也只會監控網路流量，不會監控系統，實在是很可惜，就好像在寶山前面，找不到進去的門一樣，
於是我上網找資料(我只有找中文的)，結果也找不到較明確的MRTG監控系統的方法，後來也就不了了之，
直到最近不小心看到國外的文章(我本來是想監控PVPGN線上人數，結果無意中跑去他們的網站)，
發現國外的SNMP教學實在有夠多  

雖然我知道國外資料一定很多，但是我就是抱著烏龜的心態，只想看中文的 :)，
對於英文很破的我，(近代外國荼毒我們中國人的兩樣東西，一個是鴉片，一個是英文)，
雖不想看，但是好奇心戰勝了懶惰心，不小心看了幾次，就把他們都看完啦，  

所以呢整理出最近幾天研究 SNMP 的心得，也分享給想瞭解SNMP的網友，有錯請來信指教，以免我誤人子弟  
2003.04.01 