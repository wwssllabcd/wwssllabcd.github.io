title: 安裝 PyQt 
date: 2018-05-21 01:13:59
tags: Python
toc: true
---

安裝 PyQt 
----------------
PyQT 為 python 的一款 GUI 程式, 是採用 GPL licence, 但也是不是每一版都有 windows 安裝版  
注意: 這邊可能會更推薦使用 pip 安裝, 指令為

     pip install pyqt5

pip 會自動根據 python 的版本自動匹配, 我這邊使用 `python 3.6.5` 批配到的是 `pyqt 5.10.1`

	雖然 PyQt 可以自行從官網下載並安裝, 但請務必使用 pip 安裝, 後面使用的時候問題會比較少
	
	
安裝 Pyqt5-tools
-------------------------
使用 pip 安裝 PyQt 時, 並不會把 Pyqt designr 給安裝起來, 所以使用 pip 安裝 Pyqt5-tools, 以便取得 QT designer

	pip install pyqt5-tools
	
而 designer 會在 
	
	D:\Python\Python36-32\Lib\site-packages\pyqt5-tools
	
中找到

<!-- more -->

PyQt 的 ui 檔轉換
------------
安裝好 pyqt5 後, 可以使用 qt design 來設計 UI, 把 UI 設計好了之後存檔, 會產生 .ui檔 接下來要使用此ui 檔, 我們必須把此 ui 檔案轉換成 .py檔，方便我們直接在 Python 中使用, 使用 CMD 切換到設計好的 ui 所在目錄下，執行此指令(ui檔我們取名為 myui.ui) 

    pyuic5 myui.ui -o myui.py

而 pyuic5 路徑如下( python 安裝路徑為 D:\Python36-32 為例)
    
    D:\Python36-32\Scripts\pyuic5.exe

接下來把以下文字存成'PyGui.py'  

	import sys
	from PyQt5.QtWidgets import QApplication, QDialog, QMessageBox
	from myui import Ui_Dialog
	class MyDlg(QDialog):
		def __init__(self):
			super(MyDlg, self).__init__()

            # Set up the user interface from Designer.
            self.ui = Ui_Dialog()
            self.ui.setupUi(self)

	def main_start():
		app = QApplication(sys.argv)
        window = MyDlg()
        window.show()
        sys.exit(app.exec_())


    if __name__ == '__main__':
        main_start()

再執行以下指令即可
    
    python PyGui.py 

ref:
    
http://pyqt.sourceforge.net/Docs/PyQt5/designer.html


在 QT design 中觀看 ui 預覽
------------
* 表單/預覽, 
* 或是 Ctrl+R

LineEdit 與 TextEdit 的差異
--------------
lineEdit: 單行的 edit
TextEdit: 多行


什麼是 Spacer 
---------
是當使用 vertical layout 時, 若中間不想放東西的時候用來填空的
Horizontal Spacer 水平空白条和 Vertical Spacer 垂直空白条，空白条的作用就是填充无用的空隙，如果不希望看到控件拉伸后变丑，就可以塞一个空白条到布局器里面
https://qtguide.ustclug.org/


建立一個事件
------------------

	class MyDlg(QtGui.QDialog):
	    def __init__(self, parent=None):
	        QtGui.QWidget.__init__(self, parent)
	        self.ui = Ui_Dialog()
	        self.ui.setupUi(self)
	        self.ui.btnRefresh.clicked.connect(self.chk_fun)

	    def chk_fun(self):
	        print("Good.")


直接在 MyDlg 中的 `__init__` 中加入事件, 並綁定到某個 function 就可以了, 
例如這邊看到的是一個叫`btnRefresh` 的button, 我們把這個 button 的 clicked 的事件, 綁訂到 chk_fun 這個 function , 而這個 function 印出 good

再舉一個例子, 如 combobox 的 index change 綁定事件如下
	
	myCombobox.currentIndexChanged.connect(self.cmd_idx_change)


使用 keypass event
----------------
只要在該 Qdlog 中, 複寫 def keyPressEvent(self, event): 即可, 如下所示

	def keyPressEvent(self, event):
    	key = event.key()
	    print(key)
    	super(MyDlg, self).keyPressEvent(event)

要注意的是, 如果現在 focus 的控鍵上有 keypassevent 的話, 會優先呼叫該控鍵的 event, 例如 txtedit 有自己的 page down , 所以 dialog 的不會對他造成影響


關掉 QTextEdit 的 Scoll
-----------
找到 verticalScrollBarPolicy , 並且把她設成 off 即可

設定 dialog title
----------
self.setWindowTitle

Combobox 中的下拉式 item 加長
---------------
有個屬性叫`visiable item cnt`的數字選大一點



