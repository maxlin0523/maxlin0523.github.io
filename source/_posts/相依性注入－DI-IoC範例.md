---
title: 相依性注入－DI/IoC 範例
catalog: true
date: 2021-05-30 00:05:23
subtitle:
header-img:
tags:
- C# 觀念
- DI Framework
---
# 前言
接續上篇（相依性注入－DI/IoC），這篇介紹實作的部分，關於 DI/IoC 再稍微帶過：也就是把兩個物件有相互依賴部分的控制權轉移到容器（稱為 IoC 容器），這樣的轉移行為叫做 `Inversion of Controll` ，之後在 IoC 容器裡註冊要執行依賴部分的邏輯供兩個物件來使用，這樣的註冊行為叫做 `Dependency Injection` ，日後若這兩個物件需要用到註冊的依賴物件時，則由 IoC 容器 `主動` 提供使用。

# 實作(使用 Console Application)
若現在要對世界各地籃球聯賽的戰績儲存在 Excel 裡，範例以 Nba 為例，但世界各地籃球聯賽可能有上百個，就請自行想像情境。

首先實作一些需要用到的物件：

###### Nba類別
![](https://i.imgur.com/LxvtvoW.png)


###### Excel類別
![](https://i.imgur.com/uMPFXg5.png)


###### 在 Program.cs 測試輸出
![](https://i.imgur.com/TE8a0ZU.png)


可以看到 Nba 類需要依賴 Excel 類提供的實作，也就是這兩個類別是相互依賴的關係，如果今天需求需要換成儲存在資料庫呢？
很簡單啊！實作一個資料庫類別在 Nba 類別替換實作就好了！

###### 如圖
![](https://i.imgur.com/v9n0hEd.png)
![](https://i.imgur.com/9TOr21X.png)
###### 執行後結果
![](https://i.imgur.com/cSs3vwr.png)

理所當然為：NBA is saving in the database.
但幾勒！但情境是世界各地的籃球聯賽欸！會有上百個類別要改
不就會改到中風？

我已經中風了🤬！
這時就是使用相依性注入的時機！
不知道各位有沒有發現，Excel 類別和 Database 類別都在做 DoSave 的行為，這時候請把它們萃取出介面，待會依賴注入時會用到。

如下:
###### 萃取介面
![](https://i.imgur.com/ZPWTQJi.png)
###### 繼承
![](https://i.imgur.com/9JLlCk8.png)
![](https://i.imgur.com/ieqHDWc.png)

將 Nba 類別的建構式改為外部宣告傳入
![](https://i.imgur.com/JGMrIMH.png)

這時可以來建立 IoC 容器了
請先載好以下 Nuget 套件：
* Unity
* Unity.Container


###### 在 Program.cs 實作DI/IoC 的動作

![](https://i.imgur.com/IZn81ii.png)


Done.
# 總結
如此一來，ISave 就會由注入所對應的類別取代，日後替換也相當容易，但相依性注入的功能還不只如此，若是只想要幾個籃球聯賽用 Excel 其他都用 Database 呢？那就要用 RegisterTypeFactory。

想表達的是
相依性注入的變化以及應用情境非常多，像範例的情境為建構式注入，當然還有方法注入、屬性注入、分層架構的分層隔離也要用到、還有各式擴充套件的注入．．．等，不是幾篇文就能完全理解的，這邊只是
提供一個入門的小小範例供大家參考。

範例原始碼： https://github.com/maxlin0523/DependencyInjectionSample