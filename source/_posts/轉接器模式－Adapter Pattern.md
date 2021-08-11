---
title: 轉接器模式－Adapter Pattern
catalog: true
date: 2021-06-27 00:05:23
subtitle:
header-img:
tags: 設計模式
---
# 介紹
中譯為轉接器模式，是一種結構型的設計模式( Design Pattern )，而實際上就是一個轉接器概念，將兩個彼此不相容的類別藉由一個轉接器類別轉換使彼此達到相容，進而一起完成任務。

# 現實情境
假設你有一台 PC 主機用了 `DVI` 連接螢幕，但某天螢幕壞了無法運作，幸運的是朋友最近要換螢幕所以決定把舊螢幕送你，不幸的是那個螢幕是 `HDMI` 接口，而你的主機也只能相容 `DVI` 接口
單兵如何處置？
.
.
.
可能會有以下幾種解決方式：
1. 將主機元件替換成 `HDMI` 接口的設備
2. 換一個相容 `DVI` 的新螢幕
3. 使用 `HDMI to DVI` 的轉接器
 
該如何選擇？


# 把現實情境套用到軟體設計
其實主機跟螢幕就是兩個不相容的類別，有發現什麼嗎？

方法 1：需改動到現有主機類別內部的設計
方法 2：打掉重練，等於重新設計了一個符合 `DVI` 接口的螢幕類別
上述方法很明顯都違反了 `SOLID－OCP`！

`OCP` 定義：對擴充開放對修改封閉，意思是我們應該在不修改既有程式的情況下去擴充新的功能。
看起來就只剩方法 3 了。
沒錯！它就是今天的主角 **轉接器模式( Adapter Pattern )**

# 把軟體設計情境套用到實作
首先，開啟 `Console Application` 專案
建立主機類別 `MyHost` 以及螢幕類別 `MyScreen` 

###### MyHost：只相容 DVI 接口並帶入建構式參數，設有Open方法
![](https://i.imgur.com/ONY7iqB.png)

###### MyScreen：螢幕為 DVI 規格，設有Connect方法
![](https://i.imgur.com/DTD2QAj.png)
###### IDIV
![](https://i.imgur.com/bQ6VE8B.png)

###### 輸出
![](https://i.imgur.com/2cPUxt3.png)

電腦螢幕都是正常運作狀態。

但在某天，螢幕壞了無法運作！

###### MyScreen Connect 實作改為
![](https://i.imgur.com/dasOhr5.png)

###### 輸出
![](https://i.imgur.com/IMJUpkc.png)

但經過求助後
朋友剛好有汰換掉的 `HDMI` 螢幕可以給我！

###### 增加 NewScreen 並且為 HDMI 規格，設有方法 Connect
![](https://i.imgur.com/bLApUD8.png)

###### IHDMI
![](https://i.imgur.com/iwDdeT7.png)

###### 輸出
![](https://i.imgur.com/fNxgtsA.png)

編譯失敗，原因為 `MyHost` 只能注入 `IDIV` 型別。

這邊可能會疑惑：既然 `DVI` 跟 `HDMI` 同屬螢幕又有連接 `Connect` 的行為，為何不抽出 `IScreen` 介面再注入對應的 `Screen` 類別使用呢？

這的確也是一種解決方式。

但現實工作遇到的情況會這麼容易嗎？若來源是自己不可控的第三方套件，或其他不可抗因素導致無法重構既有的架構，那麼較適合的方式就是使用 `Adapter Pattern`。

所以

###### 建立 Adapter 類別
![](https://i.imgur.com/6KigYOE.png)

`Adapter`類別說明：

* `紅底線`：代表要轉換的物件 -- Target。目標是要相容主機的 `DVI` 螢幕接口，所以要變成 `DVI` 的形狀，繼承IDIV。

* `綠底線`：代表被轉換的物件 -- Adaptee。需求是要將 `HDMI` 的接口相容 `DVI`，所以把 `IHDMI` 設為建構式參數，而在後續實作 `Connect` 方法的內部邏輯呼叫 `IHDMI Connect` ，藉此達到轉接功能。

###### 輸出
![](https://i.imgur.com/Uk8Iimw.png)


### 這樣就實現轉接功能囉！

###### 範例成員
![](https://i.imgur.com/hlcbTFM.png)

# 總結
如此一來，我們只需建立 `Adapter` 類別即可完成轉接器功能，並不會更動到原有設計，也印證了前面說的 `OCP` 原則。

範例原始碼：https://github.com/maxlin0523/AdapterPatternSample