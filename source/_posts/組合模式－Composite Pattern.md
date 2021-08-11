---
title: 組合模式－Composite Pattern
catalog: true
date: 2021-07-03 00:05:23
subtitle:
header-img: 
tags: 設計模式
---
# 介紹
中譯 `組合模式`，又稱部分整體模式，是一種結構型的設計模式（Design Pattern），主要以樹狀結構 `Tree` 的方式呈現物件之間的關係。

何謂樹狀結構？

是一種模擬現實生活中樹枝和樹葉的資料結構，例如：文件資料夾、公司組織圖、賽程表、權限．．．等具有階層關係的資料結構，最底層物件稱為「**樹葉－Leaf**」，而有任何一個子層的物件都稱為「**樹枝－Composite**」。

組合模式概念也類似鏈結串列 `LinkedList` ，每個 `ListNode` 都存有自己的值和下層 `ListNode` 的資料，且層層遞歸，所以使用場景會是一個具有階層關係的群體組織，而實作上則有分 `安全型`、`透明型` 兩種模式，兩者差別文末會再簡單說明，這篇將用透明型組合模式進行實作。

# 實作（透明型）

###### 定義部門組織圖
![](https://i.imgur.com/UTjFVy0.png)

開啟 `Console Application` 專案

###### 定義員工類別的基底類 BaseEmployee
![](https://i.imgur.com/O2iM7Sw.png)

###### 定義職位 Position 列舉
![](https://i.imgur.com/m2mVE0I.png)

###### 定義底層員工類：樹葉層 ( 不可實作 Add & Remove )，繼承 BaseEmployee
![](https://i.imgur.com/RN6mjvE.png)

###### 定義非底層員工類：樹枝層 ( 可實作 Add & Remove )，繼承 BaseEmployee ，且多了存放子層的 List
![](https://i.imgur.com/mo29Omh.png)

###### 在 Program.cs 依層級建立各個員工
![](https://i.imgur.com/Oo0Y0Vm.png)

###### 串聯出圖片上的部門組織，並將結果轉為 Json 格式
![](https://i.imgur.com/RvdrvKf.png)

###### 查看結果 （這什麼鬼？）
```
{“Subordinates”:[{“Subordinates”:[{“Name”:”Beck”,”Position”:”FrontendRD”}],”
Name”:”Albert”,”Position”:”HeadFrontend”},{“Subordinates”:[{“Subordinates”:
[{“Name”:”Minerh”,”Position”:”Intern”}],”Name”:”Duke”,”Position”:”
BackendRD”},{“Subordinates”:[{“Name”:”Minerh”,”Position”:”Intern”}],”Name”:”
Frank”,”Position”:”BackendRD”}],”Name”:”Max”,”Position”:”HeadBackend”},
{“Subordinates”:[{“Name”:”Richard”,”Position”:”DBA”},{“Name”:”Daniel”,”
Position”:”DBA”}],”Name”:”Alan”,”Position”:”HeadDatabase”}],”Name”:”Teddy”,”
Position”:”ProductManager”}
```

###### 以 Json Viewer (Notepad++擴件) 查看結果
![](https://i.imgur.com/MhiEQl6.png)

### 由此可看出樹狀結構是由最頂層物件 ( Teddy ) 涵蓋，並層層包覆。

範例原始碼： https://github.com/maxlin0523/CompositePatternSample

# 透明型 V.S. 安全型
* 透明型 ( Uniformity )：
如範例所示，可以看出每層物件都源於相同抽象類 `BaseEmployee`，優點是結構較透明、單純，所有物件都保持一致，在使用上可以相同對待每個物件，缺點是：位於樹葉層的部分其實是不需要 `Add & Remove` 方法，因為樹枝和樹葉本質上是不同東西，礙於繼承一致的抽象類所以採用例外拋出的處理方式，違反 `SOLID－ISP`，使用上較為不安全，可能會出錯。

* 安全型 ( Safety )：
基本上跟透明型相反，彌補了缺點但也損失其優點的部分，安全型則會區分成樹枝樹葉兩種元件，樹枝層給予實作 `Add & Remove` 方法，樹葉層則不允許，所以不能由相同抽象類產生，需稍微加工，結構相對不夠透明，但使用上較為安全，符合 `SOLID－ISP`。

# 參考
[設計模式系列之組合模式(Composite Pattern) — — 樹形結構的處理](https://www.gushiciku.cn/pl/pPh1/zh-tw)
[菜鳥教程-组合模式](https://www.runoob.com/design-pattern/composite-pattern.html)
