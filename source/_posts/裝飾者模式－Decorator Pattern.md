---
title: Decorator Pattern
catalog: true
date: 2021-07-11 00:05:23
subtitle:
header-img:
tags: Design Patterns
---
# 介紹
中譯裝飾者模式，是一種結構型的設計模式（Design Pattern），主要功能為動態地給一個物件附加額外的職責使物件型態保持一致，而不必衍生出其子類別，以擴展功能而言，裝飾者模式比繼承更靈活更彈性。

什麼意思呢？

假如我們有個漢堡類別，客人可以選擇加蛋或加起司或兩者都加，我們可以很直覺的設計出：繼承漢堡類而擴充出漢堡加蛋、漢堡加起司、漢堡兩者都加的子類別共三種，但如果今天食品種類多了，如三明治、蛋餅、吐司．．．等等，又或是配菜種類多了，這樣一來類別數量就會很肥大且難以維護！

若使用裝飾者模式：
你可以想像現在有個蛋蛋工廠、起司工廠，而你手上有個漢堡，若要把漢堡加蛋，就把漢堡丟進去蛋蛋工廠，噴出來的就是一個加蛋的漢堡，而加起司也一樣，依此類推，而蛋蛋工廠、起司工廠就是裝飾類別，想裝飾誰就丟誰進去，想裝飾幾層就裝飾幾層，所以裝飾者模式具有透通性 （Transparency），可以不斷的包覆下去。

# 範例
開啟 `Console Application` 專案

依介紹情境為例
###### 定義食品抽象類 BaseFood
![](https://i.imgur.com/5Epc63B.png)

###### 定義不同食品類且繼承 BaseFood
![](https://i.imgur.com/yATCe67.png)

這樣 **食品類別** 部分就搞定了，簡單吧！不過這不是重點
再來是本篇主軸 「**裝飾者模式**」


###### 定義裝飾抽象類（DecorateFood），也就是配菜部分
![](https://i.imgur.com/qVriX47.png)

裝飾類別說明
* 由建構式傳入 `被裝飾類` 方便後續進行附加額外職責的動作。
* 定義出配菜名稱、配菜價位．．．等裝飾成員。
* 繼承 `BaseFood` 並覆寫其方法使 `被裝飾類` 附加額外職責，換句話說，就是把食品類加上配菜後的名稱與價位附加上去，同時也使物件型態保持一致。

###### 定義不同配菜類且繼承 DecorateFood
![](https://i.imgur.com/jqxqgiu.png)

### 這樣裝飾部分就完成了！

假設訂單需求為漢堡兩個都加，三明治加蛋， 吐司加起士
該如何產生？

###### Program.cs
![](https://i.imgur.com/Gj88BMp.png)

###### 輸出
![](https://i.imgur.com/YMz04Dw.png)

Done.

###### 範例成員
![](https://i.imgur.com/f4KvecF.png)

專案原始碼：https://github.com/maxlin0523/DecoratorPatternSample

# 總結
回想介紹提到裝飾者模式的特性：
* 動態施加職責（需要什麼裝飾自己加）
* 物件型態一致（同屬 `BaseFood` ）
* 透通性（可以不斷包覆被裝飾物件）

都可以透過範例清楚理解，這些特性讓我們能更靈活的擴展程式碼，而使用時機為 「**當有功能相同但處理邏輯又稍有不同**」 的情況，像是後端資料快取，在功能上都是取得相同的資料但做法不同，就適合使用裝飾者模式優雅的擴展，從 `Disk I/O` ⇒ `Redis Cache` ⇒ `Memory Cache` 依序包覆，而當請求進來時，就可以從最外層 `Memory Cache` 開始查詢，有空會再實作出來分享。
# 參考
[一秒看破 裝飾者模式 Decorator Pattern](http://weisnote.blogspot.com/2013/05/decorator-pattern.html)
