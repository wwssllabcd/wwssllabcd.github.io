---
layout: post
title: "USB 列舉教學，詳解"
date: 2012-11-28 22:14
comments: true
tags: [How-To, USB]
---
當 USB 插上 HUB 時，HUB 此時正在不斷的 Polling port 的狀態，一旦偵測到電位改變時，Hub 就會通知 Host 有新的裝置，
當然會有一些USB 全速與高速的判別，這部份請參考[全速和低速識別](http://wenku.baidu.com/view/359fa76d7e21af45b307a8db.html)，就不再提了，
這邊只針對CATC所錄到的 USB 列舉作解析。

<!--more-->

Host 對 Address 0 發送 GetDescriptor(Device Descriptor) 的請求
----------
![addr0_desc](https://lh6.googleusercontent.com/-jrXS8-1fzGU/VJexGDXsXNI/AAAAAAAAstg/3YEiPcJCQuc/s0/addr0_desc.png)  

+ BYTE 0，1：80，06 代表 GetDescriptor  
+ BYTE 2，3：代表 Descriptor 索引，這裡是 1，代表 Device Descriptor  
+ BYTE 4，5：代表語言的 ID(應該是編碼， ex big5..etc ?)  
+ BYTE 6，7：代表 Length， 代表 Device 要回傳多少 Data 回來，這裡是 0x40，而 Device 傳的 Data 只能小於等於這個長度  

Device 回應 Host 所下的 GetDescriptor
----------
而 Device 回的 Device Descriptor 是
![device_desc.png](https://lh3.googleusercontent.com/-XjjyHbtCFzE/VJexHKgc9sI/AAAAAAAAsuo/aWzBf1g4RvY/s0/device_desc.png)  

+ BYTE 0 :0x12， 代表 Length， 為 0x12 個 BYTE  
+ BYTE 1 :0x01， 代表 Descriptor 的種類，01 代表 Device Descriptor  
(注：第一個BYTE為描述資訊的長度，第二個BYTE為描述資訊的類別，長度太少，host 會不接受，過常 Host 會忽略)  
+ BYTE 2，3 代表 USB 版本， value = 0200， ，也就是 USB 2.0  
+ BYTE 4 代表 Device class(裝置類別)，這裡是 0x00，代表這個 Device 的Class 是在介面描述元中定義，(如果是 0x09， 代表 Hub，參考下面的 Class code， USB-IF )  
+ BYTE 5 代表 Sub Device 類別，這裡是 0  
+ BYTE 6 代表 Protocal，這裡是 0x00 ( 如果是 0x01， 代表 Hi-speed hub with single TT( 參考各 Device 的 Class code ))  
+ BYTE 7 代表 bMaxPacketSize， 這裡是 0x40， 也就是端點 0 最大的封包大小 (非端點 1，端點 2，其他端點的封包大小由 endPoint Descriptor 描述)  
+ BYTE 8，9 代表 vid， 這裡是 0x0C76 代表 JMTek， LLC.，(如果這裡是 05E3，代表 Genesys (創唯))( VID列表 http://www.linux-usb.org/usb.ids )  
+ BYTE 10，11 代表 PID，這裡是 0x0005(0005 Transcend Flash disk) (如果是 0608，代表 USB-2.0 4-Port HUB)  
+ BYTE 12，13 代表 Deive 序號， 0x0001  
+ BYTE 14: 代表 Manufacturer 號，這裡為 1  
+ BYTE 15: 代表 Product 號，這裡為 2  
+ BYTE 16: 代表 SN 號，這裡為 0  
+ BYTE 17 numConfiguartions: Number of possible configration，這裡是 1  

注意 iManufacturer， iProduct， iSerialNumber， 這個決定了後面 GetString 請求裡 wValue 掉低字節的值和所獲得 string 的關係。  
注：如果是 Bulk-only 的裝置的話，DeviceClass， DeviceSubClass， DeviceProtocal 應全為 0  

Host 對 Address 0 發送 SetAddress 的請求
-------------------
因為每個USB Device 初始的 Address 都是 0 ，所以一旦 Host 找到了Device ，就會把他從Address 0 移開，以免跟後來新的Device衝突，
此時，Host 把這個 device 設在 Address 3 的位置
![host_set_addr.png](https://lh4.googleusercontent.com/-rut1BhE4v8g/VJexHzK-lII/AAAAAAAAstw/j9RZKLRzXwY/s0/host_set_addr.png)  

+ BYTE 0: 因為不用 Data-in ，所以不是 0x00  
+ BYTE 1: 代表 Command 種類，這裡是 0x05，代表 SetAddress ( GetDescriptor 這裡是 0x06)  
+ BYTE 2，3 : 代表 Address，範圍為 0~127   
+ BYTE 4~7: fixed 0  

Host 對 Address 3 發送 GetDescriptor 的請求
-------------------
Host 對新的 Address 下 GetDescriptor，目的可能是想要觀察 Device 有沒有正確的被配到新的位置，理論上來說，回傳的 Data 應該要一樣才對，
而接下來所有發送的位置，都會以新的位置(也就是 Address 3)來發送

Host 發送 GetDescriptor(Config類) 的請求
-------------------
再看CATC圖之前先說明一下，此時 Host 要取回 Device 的 Configuration  
而 Configuration Descriptor 中有包含好幾種 Descriptor，概述如下
+ 組態描述元(Configuration Descriptor)
+ 介面描述元(Interface Descriptor)
+ 端點描述元(Endpoint Descriptor)
+ HID描述元(HID Descriptor)
+ etc..

這邊有個問題，每支Device 的interface與endpoint的數量都不一樣，也就是說，每支 Device 的 GetDescriptor(Config類)的全部長度是不一樣的，
那 host 要叫 Device 回傳多少長度呢？

這邊 Host 把GetDescriptor(Config類)這個動作分成兩步驟  

1. 只先取 Configuration Descriptor(也就是前9 BYTE)  
2. 根據 Configuration Descriptor 中的BYTE 2,3，來得知 GetDescriptor(Config類)的總長  

請看下圖，Host 發送 GetDescriptor(Config類)，雖然整個config我不知道多長，但最起碼的Configuration Descriptor是固定9 Byte的，
所以 host 先叫device 回傳長度為9，代表Host只先取回 Configuration Descriptor
![host_GetDesc.png](https://lh6.googleusercontent.com/-Qhv7X91mAxc/VJexHUrqBPI/AAAAAAAAsts/bmrzF0ltafs/s0/host_GetDesc.png)  

Device 回傳 GetDescriptor(Config類) 的請求
-------------------
Device 回傳 9 BYTE，而前面 9 BYTE固定是 Configuration Descriptor 
![dev_configuartion.png](https://lh4.googleusercontent.com/-GwUU3I0-yas/VJexGuxtZsI/AAAAAAAAstc/oNosdyIXN28/s0/device_configuration.png)   

+ BYTE 0: 0x09， 代表這個 Descriptor 的長度
+ BYTE 1: 0x02， 代表這9 BYTE 是屬於 Configuration Descriptor 
+ BYTE 2 :0x20， wTotalLength_L，0x20(該配置返回的數據總長度，包括其下 interface， Endpoint descriptor 的總長度)
+ BYTE 3 :0x00， wTotalLength_H
+ BYTE 4: 0x01， 代表有幾個 interface，這裡為 1 (猜測， 一個USB_interface 對應一種USB邏輯設備，比如鼠標、鍵盤、音頻流。所以，在USB範疇中，device 一般就是指一個 interface。一個驅動只控制一個 interface。)
+ BYTE 5: 0x01， 代表這個 Configuration 的編號 ( Host 可由這個編號，下SetConfiguration 來做 配置的切換)
+ BYTE 6: 0x00， 代表這個 configuartion 的String Descriptor 編號，如果為 0x00，則代表沒有String Descriptor
+ BYTE 7: 0x80， 代表這個配置的一些 attribute，  Bit7 fixed 1(for historical reason), Bit6=1，代表從Usb Bus 供電， Bit5=1 代表Remote wake up
+ BYTE 8: 0x32， 所用電流，單位為 2mA，這邊代表這個裝置需要 100mA ( USB Hub 最大為 500mA)

Host 再一次發送 GetDescriptor(Config類) 的請求
-----------------------
雖然 Device 告訴 host 總長為0x20, 但這裡 Host 很闊氣的下了0xFF
![Host_getConfiguration.png](https://lh6.googleusercontent.com/-cv3YQWeJ0sg/VJexE2l09RI/AAAAAAAAss8/2Kq_v7-I-zc/s0/Host_getConfiguration.png)  

Device 再一次回傳 GetDescriptor(Config類) 
------------------- 
![device_configuration.png](https://lh4.googleusercontent.com/-GwUU3I0-yas/VJexGuxtZsI/AAAAAAAAstc/oNosdyIXN28/s0/device_configuration.png)  

這裡面一共有四組 (可藉由計算長度 (第一個BYTE) 來判斷)

+ 第一組為 Configuration，共 9 BYTE，而 0x02 代表 組態描述元(Configuration Descriptor)
+ 第二組為 Interface    ，共 9 BYTE，而 0x04 代表 介面描述元(Interface Descriptor)
+ 第三組為 Endpoint     ，共 7 BYTE，而 0x05 代表 端點描述元(Endpoint Descriptor)
+ 第四組為 Endpoint     ，共 7 BYTE，而 0x05 代表 端點描述元(Endpoint Descriptor)

根據USB規定，當指令要取得組態描述元時，則裝置必須要將裝置描述元，介面描述元，HID描述元，端點描述元的資料全都回傳給主機，並且按照規定的順序排列  
而 Configuration 前面已經有講過了  
接下來會剖析 介面描述元(Interface Descriptor) 與 端點描述元(Endpoint Descriptor)  

Device Configuartion 中的介面描述元(Interface Descriptor)
-----------------------
以下這張圖同 device_configuration，貼在這邊只是方便對照，interface Descriptor 請從 offset 0x9 開始看起
![device_configuration.png](https://lh4.googleusercontent.com/-GwUU3I0-yas/VJexGuxtZsI/AAAAAAAAstc/oNosdyIXN28/s0/device_configuration.png)  

+ BYTE 0: 0x09， 代表長度
+ BYTE 1: 0x04， 代表 Interface descriptor
+ BYTE 2: 0x00， Interface Number
+ BYTE 3: 0x00， AltemateSetting，代表交替設定 (linux 會有機會交替的使用不同的AltemateSetting)
+ BYTE 4: 0x02， 代表此介面所使用的端點數 (對應到 linux unsigned num_altstting)
+ BYTE 5: 0x08， Class Code (群組代碼)，代表Interface Class， 0x08 代表 mess storage( see http://www.usb.org/developers/defined_class)
+ BYTE 6: 0x06， SubClass code(次群組代碼)：主群組代碼如果有細分的話，就在這裡描述，而 0x06 代表SCSI ( 0x02 代表CD-ROM)(對應到 linux kernel drivers/usb/storage/protocol.h)
+ BYTE 7: 0x50， interfaceProtocol(介面協定), 0x50 代表Bulk-Only Transfer( 對應到 linux kernel ，drivers/usb/storage/transport.h)
+ BYTE 8: 0x00， Interface 的 String Descriptor， 0x00 代表沒有

Mass Storage specifications Overview 節錄 (對應到BYTE 6) 
![Usb_Spec_SubClassCode.png](https://lh4.googleusercontent.com/-yeF9Nj3nCK4/VJexFkhy-rI/AAAAAAAAstI/6yhtkbkwa6A/s0/Usb_Spec_SubClassCode.png)  

Mass Storage specifications Overview 節錄 (對應到BYTE 7) 
![Usb_Spec_MS_trans_proto.png](https://lh5.googleusercontent.com/-5nnyc8I2d1E/VJexFQgixFI/AAAAAAAAst0/vX2M8_qIE8M/s0/Usb_Spec_MS_trans_proto.png)  

Device Configuartion 中的端點描述元(Endpoint Descriptor)
-----------------------
共 7 BYTE，描述端點的屬性及位置 (不描述端點 0)，端點的數目是由 Interface descriptor 的BYTE 4 得知每個端點在Device 中，都是代表某個記憶體位置，而各個端點有不同的傳輸型態及最大傳輸值  

PS: 以下這張圖同 device_configuration，貼在這邊只是方便對照，Endpoint Descriptor 請從 offset 0x12 開始看起
![device_configuration.png](https://lh4.googleusercontent.com/-GwUU3I0-yas/VJexGuxtZsI/AAAAAAAAstc/oNosdyIXN28/s0/device_configuration.png)  

+ BYTE 0: 0x07， 代表長度
+ BYTE 1: 0x05， Descriptor Type，0x05 代表端點描述元
+ BYTE 2: 0x81， EndPoint Address， BIt[3:0] 代表端點號，， BIt[7] 1:In， 0: out，所以 0x81 代表 Endpoint 1， 負責 data-In，而控制端點是雙向的，所以不用看BIt7
+ BYTE 3: 0x02， Attribute(端點屬性): Bit[1:0] 代表支援的傳輸模式，10b(即 2) 代表Bulk 類
+ BYTE 4: 0x00， wMaxPacketSize_L: 最大傳輸量，如果是HighSpeed，則是 512，也就是 0x200
+ BYTE 5: 0x02， wMaxPacketSize_H ( 在 linux 中，kernel 會根據 wMaxPacketSize 的大小，使用 kalloc 配置一塊記憶體給該 endptr 使用而有趣的是，由於是LSB的關係，linux 會使用一個 le16_to_cpu 函數把 0x0002 轉成 0x0200)
+ BYTE 6: 0xFF， Interval， 代表多少間隔內，Device 最多只發一次NAK封包 (或是在中斷傳輸中，host 間隔多少時間來 polling device)

Host 下 Get String Descriptor
-----------------------
Host Get string descriptor(wValue MSB: Descriptor Type =0x3)
![Host_GetStringDesc.png](https://lh5.googleusercontent.com/-natnFwbreG4/VJexE-_uiyI/AAAAAAAAssw/AE_sM27E-RI/s0/Host_GetStringDesc.png)  

+ BYTE 2: index，這裡的 index=0， 代表只取LanguageID
+ BYTE 3: 代表GetString Descriptor
+ BYTE 4，5 : 代表Language ID， 這裡為 0，代表沒有特別的ID
+ BYTE 6，7: 代表Length

Device 回傳 String Descriptor
------------------------
![Device_strDesc.png](https://lh6.googleusercontent.com/-w0uP5CSA9lA/VJexDYqmG2I/AAAAAAAAssk/dqg0RB5vJeg/s0/Device_strDesc.png)  

+ BYTE 0: Length， 長度為 4
+ BYTE 1: DescriptorType， 為 03
+ BYTE 2，3: bString， 0409， 代表Language ID = 0x0409，代表使用的Language 為 English (United States)  

PS: 有關 Lang ID ，可見USB Language Identifiers(LANGIDs).PDF

Host 再下一次 GetString Descriptor ( Get Product String )
--------------------
接下來，Host 會在下一次GetString Descriptor 下，與上次不同的是，wValue 的LSB(BYTE 2), 也就是Descriptor Value 變成了 2
![Host_GetStrDesc_2.png](https://lh4.googleusercontent.com/-K0nIwPO37e8/VJexDZxXFnI/AAAAAAAAssc/fChoznfx5ps/s0/Host_GetStrDesc_2.png)  

這裡 wValue 低八位 =2， 對應Product String.  
而 wIndex，也就是Language ID變成了 0x0409( Byte 4，5)  

Device 回傳 GetString Descriptor ( Product String )
------------------------
![Device_ProductStr.png](https://lh6.googleusercontent.com/-Rsv-Rw29cjc/VJexDUzxlwI/AAAAAAAAssg/XiL5QUAPin0/s0/Device_ProductStr.png)  

把這些數據按照ASCII編碼翻譯過來，就是現在 Host 端的 device name 了
![ProductStr_ascii.png](https://lh5.googleusercontent.com/-1zYP8d87Gac/VJexEx2t85I/AAAAAAAAsu0/KpX1k_lDH8o/s0/ProductStr_ascii.png)  

到此基本上 USB Emulation就已經結束了，接下來就是UFI 的 inquriry 了