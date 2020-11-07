title: Bitcoin 簡介
date: 2015-08-21 23:32:31
tags: [Bitcoin]
---
發明者: 中本聰([1])，但這只是化名，真實身份不明，應該是一群想撈錢的集團幹的

經過: It's a very long story，請見資料([2])

使用過程
----------
其實就是妳的電子錢包，把交易的訊息(如我要把 100 個 bitcoin 轉給 A)這條訊息用數位簽章簽過，
經過P2P網路送給所有的錢包使用者手上，利用P2P網路把交易的訊息傳遞出去，利用RSA保證身份，是一種去中心化的貨幣

產生比特的方式
-------------
就是所謂的 mining(挖礦)，透過這個公式 

	SHA256( Block data + Random nonce ) < 難度值

來算出來([3])，而 Block data 就是上一筆交易的資料，也就是說，Block data 再加上某個值 N ，如果丟到 SHA 中運算小於這次難度值的結果的話就代表挖到礦，
挖到的東西就是新的 block，這個Block會把數字 N 還有這段期間收到的交易資料還有新的難度值，包成一包變成新的BlockNo後，接在舊的後面後，再藉由P2P傳出去，
別人在根據新 BlockNo 繼續玩上面那套公式，而所謂的"難度值"會根據 BlockNo 根據當初設計的公式調整(這個數值會每隔2016個block，
網絡大約每小時創建6個塊，創建2016塊大約2週)調整一次)，
所以後面會越來越難挖，變成總數(發行量)會趨近於某個數值，又因為交易的資料是伴隨著新的 Block 用P2P的方式散布出去，
所以有時候交易的資料並不會馬上顯示出來，約要等 6 個 block 左右的時間才可能散布到所有使用者手上

而[SHA-256]是一種[雜湊演算法] (Hash algorithm)，他的公式會讓輸入的值變成長度固定的數字，
例如輸入 Fox，可能會產生 DFCD3454([4])，他的原理有點類似拿一個質數來取餘數，如以質數 13 來做雜湊的 Base Key 的話，
數字131的雜湊值就是1( 131 mod 13 = 1，就是他的餘數)，而 1 的雜湊值也是 1，
而 261 的雜湊也是 1，發生了有好幾個數值Hash都相同，這個就叫做碰撞，產生碰撞的雜湊代表 Base Key 選的很爛，
通常要大一點的質數才行，而SHA演算法就是以數學的角度上去確定這個碰撞的機會很小(所以才叫 "Secure" Hash Algorithm )


至於怎樣確定 SHA 是安全的，本人密碼學上課都在神遊，so ... 就到此為止



[1]:https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%9C%AC%E8%81%AA
[2]:https://yowureport.com/%E5%85%A9%E5%B9%B42%E8%90%AC%E5%80%8D%E7%9A%84%E5%8D%87%E5%80%BC%EF%BC%8C%E6%9C%80%E5%88%92%E7%AE%97%E7%9A%84%E6%8A%95%E8%B3%87-%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3-bitcoin/
[3]:https://www.ptt.cc/bbs/Soft_Job/M.1385557793.A.5E6.html
[4]:https://zh.wikipedia.org/wiki/%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B8#/media/File:Hash_function.svg

[SHA-256]:https://zh.wikipedia.org/zh-hant/SHA%E5%AE%B6%E6%97%8F

[雜湊演算法]:https://zh.wikipedia.org/zh-hant/%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B8