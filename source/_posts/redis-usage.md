---
title: C# 串接 Redis
catalog: true
date: 2022-06-15 03:01:46
subtitle:
header-img:
tags:
- Redis
- C#
---
## 前言

先前 [Docker 佈署 Redis](./2021/10/09/redis-in-docker/) 記錄如何佈署 Redis，這篇來記錄如何在 C# 串接 Redis 資料庫和一些基本的資料結構使用

## 介紹
Redis 是一個 open source 且 in-memory 的 NoSQL 資料庫( key-value 類型 )
* **特點**
    * 支持持久化，可將內存資料存於硬碟，備份模式如：`RDB`、`AOF`
    * 豐富資料類型，如：`String`、`List`、`Set`、`ZSet`、`HashSet`
    * 支持高可用及高擴充 Solution，如：Master-Slave(主從)、Sentinel(哨兵)、Cluster(叢集)模式
    * 性能極高，特性：I/O Multiplex、In-memory 
    * 豐富功能，如：pub/sub 訂閱、data expired
* **應用場景**
    * 具有`時效性`的資料控制，如：驗證碼、Token
    * 分散式架構資料共享，如：快取
    * 即時訊息查詢，如：在線人數、廣播訊息

## 下載套件與串接

* 下載 `StackExchange.Redis` 套件
![](https://i.imgur.com/86mSmgi.png)

* 串接
※ Redis 使用多路複用(Multiplex)，也就是單一個執行緒可以同時處理多個連線，所以沒有 connection pool 的情況，不需使用 using 來建立 & 釋放
```C#=
// 連線字串
string address = "127.0.0.1:6379";
// 建立連線，不需 using
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(address);
// 取得實體
IDatabase db =  redis.GetDatabase();
```


## String

```c#=
// 設定資料
var stringKey = "Hello";
await db.StringSetAsync(stringKey, "0523");

// 取得資料
string value = await db.StringGetAsync(stringKey);

// 刪除資料
await db.KeyDeleteAsync(stringKey);
```

## List

value 為一個 List 結構，且 Redis 的集合 Index 都是從 1 開始

```c#=
var listkey = "List";
// 從頭部加入元素
await db.ListRightPushAsync(listkey, "AAA");

// 從尾部加入元素
await db.ListLeftPushAsync(listkey, "BBB");

// 取出指定 Index 元素
var indexValue = await db.ListGetByIndexAsync(listkey, 1);

// 取出陣列
var array = (await db.ListRangeAsync(listkey)).ToStringArray();

// 取出子陣列，取 index 1 之後的 2 個
var subArray = (await db.ListRangeAsync(listkey, 1, 2)).ToStringArray();

// 插入多筆資料
var insertCount = await db.ListRightPushAsync(listkey, new RedisValue[] { "B", "B", "B", "D" });

// 刪除特定資料，將 A 的資料全部刪除
await db.ListRemoveAsync(listkey, "A", 0);

// 刪除特定資料，從頭開始刪除 1 筆 B 的資料
await db.ListRemoveAsync(listkey, "B", 1);

// 刪除特定資料，從尾開始刪除 2 筆 B 的資料
await db.ListRemoveAsync(listkey, "B", -2);

// 保留一段範圍的資料，取 index 1 之後的 2 個
await db.ListTrimAsync(listkey, 1, 2);
```

## Set
value 為一個 Set，且為一個 `不重複` 的集合

```C#=
var setKey = "set";

// 建立一筆資料
await db.SetAddAsync(setKey, "1");

// 建立多筆資料
await db.SetAddAsync(setKey, new RedisValue[] { 1, 1, 2, 3 });

// 刪除指定元素 1 的資料
await db.SetRemoveAsync(setKey, 1);

// 取得 Set 長度
var setLength = await db.SetLengthAsync(setKey);

// 取得 Set 所有元素
var setItems = await db.SetMembersAsync(setKey);

// 判斷指定元素 1 是否存在
var itemExists = await db.SetContainsAsync(setKey, 1);
```

## ZSet(Sorted Set)
value 為一個 SortedSet，為一個 `排序且不重複` 的集合，寫入時會依照 Score 進行排序

```C#=
var sortedSetKey = "sortedSet";
// 新增一筆資料
await db.SortedSetAddAsync(sortedSetKey, "Amy", 5);

// 新增多筆資料
await db.SortedSetAddAsync(sortedSetKey, new SortedSetEntry[]
{ 
    new SortedSetEntry("Andy", 8),
    new SortedSetEntry("Danny", 4)
});

// 取得 SortedSet 長度
var sortedSetLength = await db.SortedSetLengthAsync(sortedSetKey);

// 取得 SortedSet 所有元素
var sortedSetItems = await db.SortedSetRangeByValueAsync(sortedSetKey);

// 取得 SortedSet 所有元素包含分數
var sortedSetItemsAndSorce = await db.SortedSetRangeByScoreWithScoresAsync(sortedSetKey);

// 取得分數遞減排序後 Danny 的排名，默認從索引 0 開始排
var rankDanny = await db.SortedSetRankAsync(sortedSetKey, "Danny", Order.Descending);

// 取得分數遞減前10名元素的排名，默認從索引 0 開始排
var rankValues = await db.SortedSetRangeByRankAsync(sortedSetKey, start: 0, stop: 10, order: Order.Descending);

// 取得分數遞減前10名元素的排名和分數，默認從索引 0 開始排
var rankValueAndScore = await db.SortedSetRangeByScoreWithScoresAsync(sortedSetKey, start: 0, stop: 10, order: Order.Descending);

// 刪除指定元素
await db.SortedSetRemoveAsync(sortedSetKey, "Andy");
```

## HashSet
value 為一個像是 C# Dictionary 結構

```C#=
var hashKey = "hashData";

// 新增資料
await db.HashSetAsync(hashKey, new HashEntry[]
{
    new HashEntry ("Number", "1"),
    new HashEntry ("Name", "Max"),
    new HashEntry ("Phone", "3345678")
});

// 查看指定資料的所有值
var hashSetValues = await db.HashValuesAsync(hashKey);

// 查看指定資料的所有 Key
var hashSetKeys = await db.HashKeysAsync(hashKey);
```

## 資料過期(data expired)

```C#=
// 過期時間
var expiredTime = TimeSpan.FromSeconds(5);

// 加入測試資料
await db.StringSetAsync("sample", "hello world");

// 設定過期時間
var success = await db.KeyExpireAsync("sample", expiredTime);
```

[原始碼](https://github.com/maxlin0523/redis-usage)

## 參考
[Redis系列 - C#存取Redis](https://jed1978.github.io/2018/05/11/Redis-Programming-CSharp-Basic-1.html)

[.NET 6使用Redis](https://www.cnblogs.com/Lulus/p/16004541.html)
