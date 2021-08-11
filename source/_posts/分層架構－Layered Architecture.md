---
title: 分層架構－Layered Architecture
catalog: true
subtitle: 
date: 2021-05-16 00:05:23
tags: 設計模式
header-img:
---
# 介紹
一般分為 `Controller`、`Service`、`Repository` 三層。各層之間依賴介面，符合 SOLID－DIP，只需要針對各層介面提供的方法去使用並達到 `相依性注入`，而每層間參數物件傳遞也需進行隔離，如此一來也達到 `關注點分離`，大家一起分工各層時也不會互相影響。

# 定義
* **Controller（展示層）**：主要負責處理來自於用戶端的 `請求與回應 I/O`。接收 http 請求後呼叫 `商業邏輯層` 進行後續處理，之後回應結果到用戶端。

* **Service（商業邏輯層）**：主要負責處理 `商業邏輯` 運算。商業邏輯處理完後呼叫 `資料存取層` 進行後續處理，再回應結果到展示層。

* **Repository（資料存取層）**：主要負責資料存取。專門處理 `資料存取` 的邏輯，例如：使用 Dapper 的 Sql 語法拼裝、取得資料庫連線、串接 API 服務．．．等，存取後再回應結果到 `商業邏輯層`。

* **Common（共用層）**：各層都會參考。將各層都共用的物件或是擴充方法都存放於此。

# 專案架構圖
![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F80c04083-a5be-4f2f-877e-56b616066946%2FUntitled.png?table=block&id=caaa0ab9-e2aa-4ebf-9b50-749f0a40e2a1&spaceId=59d38cce-92ff-4673-85e7-4a3082371d03&width=4270&userId=6c123588-4407-4a28-9aae-2bb53ad5b808&cache=v2)

# Dto、Enity 補充
你可能會疑惑 Dto、Enity 的部分，其實就是我對每層間參數物件的命名，若想叫 Apple、Banana 也可以，重要是它在分層架構的職責是什麼？職責是為了達到各層之間 `參數物件隔離` 的動作，例如：我刪除了 Dto 的一個屬性，之後轉換到 Enity 也不會影響到屬性數量，因為兩者已經做了隔離的動作，若這邊不進行隔離，參數一條龍傳下去時，我可能在上層參數刪除一個屬性，那麼就會一路影響到 `Repository` 之後串接到資料庫，若資料表欄位沒同步處理就會造成例外產生；若你的同事正在修改 `Repository` ，你改了 `Controller` 的參數屬性，若沒進行參數隔離，那麼你的同事就會被影響。

總之，請想像每層都是獨立的空間，當任一層有改動時，都不會影響到其他任何一層，詳細可以參考後續的實作範例。

# 總結
以前做完此架構專案時，會疑惑為何要分如此多層，直到複雜度慢慢變多時才漸漸理解好處，寫起來會特別有安全感。
如果目前還不是很理解的話，就我的經驗來建議，就先硬著頭皮學著做，知道個大概的觀念！等到實際維護複雜度較高的專案才比較快能體會分層架構的好處（前提是專案要有使用分層）。

範例原始碼：
https://github.com/maxlin0523/WebAPI

# 參考
[隨手 Design Pattern (2) - 軟體分層設計模式 (Software Layered Architecture Pattern)](https://raychiutw.github.io/2019/%E9%9A%A8%E6%89%8B-Design-Pattern-2-%E8%BB%9F%E9%AB%94%E5%88%86%E5%B1%A4%E8%A8%AD%E8%A8%88%E6%A8%A1%E5%BC%8F-Software-Layered-Architecture-Pattern/)