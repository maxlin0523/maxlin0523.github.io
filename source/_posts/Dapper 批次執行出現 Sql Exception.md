---
title: The sql exception occurred in Dapper's batch execution
catalog: true
date: 2021-07-07 00:05:23
subtitle:
header-img:
tags: 日常除錯
---
# 問題描述
來源是份爬蟲的 Json 物件，而此資料會 post 到 API 服務最終存入 DB，資料型態為 List 且物件屬性有 30 幾個，今天在測試時， post 有 800 多筆，但 API 回傳的儲存筆數卻總是 0 。

經過一段時間勘查，問題出在使用 Dapper 新增整包 List 進 DB 的過程出現例外，但 Exception Filter 似乎沒把 `Sql Exception` 包進去，所以回傳了預設值。

於是乎！我在 Insert 方法包了一層 Try Catch 後，確實有抓到「**字串或二進位資料會被截斷**」的 `Sql Exception` 訊息，那既然知道例外是長度超出限制而產生，問題又來了，哪一筆的哪一個欄位發生的？總不可能 800 多筆每筆 30 幾個欄位逐一去對照吧！又或是把 Table 欄位長度限制都開到 MAX 後當作沒發生。
# 處理方式
可以把範圍限縮在找出有問題的那筆，於是我把 Dapper 執行的 Sql Command 字串改為前後包上 Try Catch 且成功新增回傳 `success` ，例外則回傳 `ErrorMessage`。
```sql=
BEGIN
	BEGIN TRY
		/* ***********************************************************************
		**               原本的Sql Command                                      **
		*************************************************************************/
		SELECT 'success'
	END TRY
	BEGIN CATCH
		SELECT ERROR_MESSAGE()
	END CATCH
END
```

再把原本的 Insert 方法內改寫成 `foreach` 逐筆執行並回傳 `string`，這樣一來，List 內每筆的執行狀況都能清楚知道是否成功，接下來新增一個爬蟲物件的 List，把不是回傳 `success` 字串的物件加入並設中斷點查看，即可找出有問題的資料，接續找到長度超出範圍的欄位。