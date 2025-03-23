---
title: MongoDB Read Preference
catalog: true
date: 2023-11-21 00:15:40
subtitle:
header-img:
tags:
- MongoDB
---
MongoDB 提供了不同的 Read Preference(讀取偏好)的模式，用來決定 Replica Set 架構下 `讀的操作`

有以下幾種模式:

## primary
一切`讀的操作`都是由 Primary 節點進行

## primaryPreferred 
正常狀況下`讀的操作`都是由 Primary 進行, 當 Primary 故障則是替換由 Secondary 節點進行

## secondary
一切`讀的操作`都是由 Secondary 節點進行

## secondaryPreferred
正常狀況下`讀的操作`都是由 Secondary 進行, 當 Secondary 故障則是替換由 Primary 節點進行

## nearest
從延遲最小的節點進行`讀的操作`, 不管該節點是 Primary 或 Secondary。

---

用哪一種取決於不同場景和需求，像是：`金錢交易` 或 `賠率推送` 等對資料的即時性和一致性有較高要求的情況用 primary 模式、需要 `快速呈現` 數據的場景則用 nearest，而讀寫分離通常使用 Secondary or SecondaryPreferred 用來減輕 Primary 的負擔

![](https://i.imgur.com/CJqwaPS.png)

# 參考
[MongoDB 事务 —— 基础入门篇](https://zhuanlan.zhihu.com/p/107278045)
