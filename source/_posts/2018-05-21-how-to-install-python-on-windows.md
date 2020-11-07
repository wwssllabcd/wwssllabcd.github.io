title: Python windows 安裝, 心得, 教學
date: 2018-05-21 01:13:59
tags: Python
toc: true
---

下載並安裝 Python
----------------------

請至 [Python windows 下載頁面](https://www.python.org/downloads/windows/), 不是每個版本都有 window 的安裝版

1. 最好選擇 Python 3.x, 因為選 2.7 會有檔名多國語言問題, dos 下讀檔會亂碼, py 3 就沒有這問題  
2. 最好選 32bit 的, 因為如果要打包成單一執行檔(exe file), 打包完在 32 bit 的環境跑不起來, 且有 include dll 批配的問題  
3. 要選 32bit 還是 64 bit, 基本上要看你用到的 DLL 決定, 例如你有些額外的 dll 是使用 w32 的, 那基本上你使用 64bit 的 ptyhon 就不行, 使用而且 64 bit dll 還有 ctype call address 的問題, 建議如果不想搞死自己, 那就最好是選 32bit 的比較保險

這邊是選 [Python 3.6.5](https://www.python.org/ftp/python/3.6.5/python-3.6.5.exe) 下載
	   
安裝時請注意以下幾點

* 請注意安裝路徑, 他預設是在"使用者"目錄下面, 最好換到非中文目錄底下
* 要移除時, 必須執行安裝程式後, 裡面有個uninstall, 在 window 那邊好像找不到移除方式
* 安裝時選 customize install, 這樣才可以自選安裝路徑
* 也順便選 Add python 3.6 to path

<!-- more -->

安裝 pip
-------------------
如果是安裝 python 3.5 以上的, 都會預設安裝 pip, 所以只要更新 pip 即可, 這邊舉出了舉多種 python 取得 pip 所對應的方法, 分述如下:

* Python 3.4 以前的版本: 取得 get-pip.py 後, 放到 Python 安裝目錄下後, 執行

		python get-pip.py   
 
pip 會建立在例如 `D:\Python27\Scripts` 之下, 請把 `get-pip.py` 這個 script 加入到 path 中


* Python 3.6: 是內建pip的, 所以要使用的時候, 直接打開 dos cmd 輸入 pip 指令即可更新 pip (前面的 python 不能省略)    
 
		python -m pip install --upgrade pip

更新某個套件也可以用 pip, 若要更新 pyqt5 時, 指令如下
    
	pip install -U pyqt5

其他指令可藉由 `pip -h` 查到

關於 pip 與 pip3 的差異
----------------
pip 和 pip3 都在 Python36\Scripts\ 目錄下, 如果同時裝有python2 和 python3, pip 默認給 python2 用, pip3 指定給 python3 用, 如果只裝有 python3，則pip和pip3是等價的, 安裝了python3之後，就會有pip3
