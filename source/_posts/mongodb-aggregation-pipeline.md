---
title: MongoDB Aggregation Pipeline
catalog: true
date: 2022-06-08 01:22:29
subtitle:
header-img:
tags:
- MongoDB
---
## 前言

`Aggregation 聚合` 主要用於統計篩選資料，以 `pipeline` 的型式分成多個 `stage`，後面的 `stage` 會拿前面的產出結果當成輸入資料，每個 `stage` 做一次資料的聚合。

可以將每個步驟看成一個個 `stage`，而整個流程就是一個 `Aggregation pipeline`

![](https://i.imgur.com/HSVB1YO.jpg)

## 基礎表達式

簡單介紹 Aggregation 一些常見的表達式：

| 表達式   | 功能                       |
|:-------- |:-------------------------- |
| $project | 定義回傳的 document 結構等 |
| $match   | 過濾篩選資料               |
| $limit   | 限制回傳 n 筆              |
| $skip    | 略過 n 筆資料              |
| $group   | 資料分群                   |
| $sum     | 計算總和                   |
| $avg     | 計算平均值                 |
| $sort    | 排序                       |


首先，備好我們要測試的資料：

![](https://i.imgur.com/OeJroxB.png)

### $project

* 將要取得的欄位賦予 `1`

`db.user.aggregate({"$project": { "userName" : 1, "age" : 1 }}).pretty()`

![](https://i.imgur.com/30q9Hvd.png)

* 將不要的欄位賦予 `0`

`db.user.aggregate({"$project": {  "age" : 0 }}).pretty()`

![](https://i.imgur.com/Y1nQ0bM.png)

※ 不可混用 `0` 或 `1`

* 產生自定義欄位

`db.user.aggregate({ "$project": { "userDescription" : { "$concat" : [ "$userName", "-", "$gender" ]}}}).pretty()`

![](https://i.imgur.com/a2VWk4j.png)


### $match

篩選資料

`db.user.aggregate({"$match": {  "age" : { "$lt" : 30} }}).pretty()`

![](https://i.imgur.com/b22Ds2j.png)

### $limit

限制資料回傳的筆數

`db.user.aggregate({"$limit": 3 }).pretty()`

![](https://i.imgur.com/rL2Rvmm.png)

### $skip

略過 `5` 筆資料


`db.user.aggregate({"$skip": 5 }).pretty()`

![](https://i.imgur.com/kTN2DYQ.png)

### $group

使用時必須要定義 `_id`

`db.user.aggregate({ "$group" : { "_id" : "$gender" }})`

![](https://i.imgur.com/zyHdrs2.png)

通常搭配 `$sum` & `$avg` 等聚合表達式來做使用

`db.user.aggregate({ "$group" : { "_id" : "$gender", "totalAge" : { "$sum" : "$age" }, "avgAge" : { "$avg" : "$age"} }})`

![](https://i.imgur.com/QxybVnS.png)


## Aggregation Pipeline

使用方式：

`db.collection.aggregate( [ { <stage1> },　{ <stage2> }, { <stage3> }, ... ] )`

假設情境為：
1. 篩選 25 歲以上的使用者(stage1)
2. 將使用者依年齡遞減排序(stage2)
3. 取得排序前5位使用者(stage3)
4. 算出平均年齡(stage4)

`db.user.aggregate([ { "$match" : { "age" : { "$gt" : 25 } } }, { "$sort" : { "age" : -1  } }, { "$limit" : 5 }, { "$group" : { "_id" : null, "top5_avgAge" : { "$avg" : "$age"  } } }   ])`

顏色用以區分各個 stage，對比上面的步驟，就可以清楚知道每個 stage 的作用。

![](https://i.imgur.com/xq8TbUU.png)

## 補充 $facet

用來一次執行多個 aggregation pipeline

使用方式：

```
db.collection.aggregate({ $facet:
   {
      <aggregation_pipeline_1>: [ <stage1>, <stage2>, ... ],
      <aggregation_pipeline_2>: [ <stage1>, <stage2>, ... ],
      ...
   }
})
```

假設情境為：

1. 第一個 aggregation pipeline 為取年齡小於 30 的使用者
2. 第二個 aggregation pipeline 為取出所有使用者的暱稱

`db.user.aggregate({ "$facet" : { aggregation1 : [ { "$match" : { "age"　:{ "$lt" : 30 } } } ] , aggregation2: [{ "$project" : { "userName" : 1 }  }   ] }})`

![](https://i.imgur.com/8R5wF3Z.png)

## 參考

[Aggregation Pipeline — MongoDB Manual](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/)

[MongoDB 聚合 - 菜鸟教程](https://www.runoob.com/mongodb/mongodb-aggregate.html)
