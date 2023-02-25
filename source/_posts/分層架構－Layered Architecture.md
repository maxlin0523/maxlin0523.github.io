---
title: Software Layered Architecture
catalog: true
subtitle: 
date: 2021-05-16 00:05:23
tags: 設計模式
header-img:
---
### 介紹
一般分為 `Controller`、`Service`、`Repository` 三層。各層之間依賴介面，符合 SOLID－DIP，只需要針對各層介面提供的方法去使用並達到 `相依性注入`，而每層間參數物件傳遞也需進行隔離，如此一來也達到 `關注點分離`，大家一起分工實作層需求時也不會互相影響，更可以方便進行隔離測試。

### 分層定義
* **Controller（展示層）**：專注處理來自於客端的 `請求與回應 I/O`。接收到請求後呼叫 `商業邏輯層` 進行後續處理，之後回應結果到客端。

* **Service（商業邏輯層）**：專注處理 `商業邏輯` 運算。商業邏輯處理完後呼叫 `資料存取層` 進行後續處理，再回應結果到展示層。

* **Repository（資料存取層）**：主要負責資料存取。專注處理 `資料存取` 的邏輯，例如：使用ORM 對資料庫存取、串接其他 API 服務．．．等行為，做完後再回應結果到 `商業邏輯層`。

* **Common（共用層）**：各層都會參考。將各層都共用的物件或是擴充方法都存放於此。

### 分層架構圖
![](https://i.imgur.com/gTOngyn.png)


### 參數傳輸物件說明
* **Entity** 
> 作用在 Repository 層，從資料庫獲取資料後的第一層轉型物件，基本與 Select 欄位一致，或是當作傳入條件式參數之類...的物件，這邊是作為對 Repository 層調用的傳入與傳出
* **Dto**
> 作用在 Service 層，全稱`資料傳輸物件 Data Transfer Object`，這邊是作為對商業邏輯層調用的傳入與傳出，例如：一個 Service 層介面方法，可能會調用多個 Repository 層方法去做商業邏輯演算，而最後產出的資料就可以封裝層一個 DTO 再傳出給後續的 layer。

透過這樣的隔離方式，就能保證各層間是相互獨立、互不影響，溝通時只針對各層的介面去調用，也可以更好的去進行合作開發或測試。

### 總結
以前做完此架構專案時，會疑惑為何要分如此多層，直到複雜度慢慢變多時才漸漸理解好處，寫起來會特別有安全感。

如果還不是很理解的話，就我的經驗來建議，就先硬著頭皮學著做，知道個大概的觀念！等到實際維護複雜度較高的專案才比較快能體會分層架構的好處（前提是專案要有使用分層）。

# 參考
[隨手 Design Pattern (2) - 軟體分層設計模式 (Software Layered Architecture Pattern)](https://raychiutw.github.io/2019/%E9%9A%A8%E6%89%8B-Design-Pattern-2-%E8%BB%9F%E9%AB%94%E5%88%86%E5%B1%A4%E8%A8%AD%E8%A8%88%E6%A8%A1%E5%BC%8F-Software-Layered-Architecture-Pattern/)