---
title: Redis 腳本
catalog: true
date: 2021-12-11 03:45:00
subtitle:
header-img:
tags:
- Redis
- Lua
---
Redis 腳本是由 Lua Script 搭配 redis 的 call 指令來完成與 server 端的溝通邏輯，主要好處：
1. 可由腳本達到一次處理多條指令的效果，減少指令`來回往返時間`(Round-trip delay time)
2. 支援`原子性`操作
3. 多個指令間可保有`關聯性`，由此組合出更多樣化的功能
4. `序列化執行`，可當作交易使用

要注意的是：由於 Redis 腳本是`序列化執行`，當某個腳本執行時間太久，則會引起 server 的阻塞。

## 腳本使用方式

Redis 腳本主要以 key-value 形式儲存在 server 端內存，其中 `value` 為 Redis 腳本並且是 Lua 語法，而 `key` 為該 Redis 腳本經 SHA1 雜湊演算出來的值。

#### EVAL `<script>`

執行腳本取得結果並同時儲存在內存，若已存在則忽略不儲存

* **腳本無傳入參數**

若無傳入參數，指令後要補 0

![](https://i.imgur.com/3PaGPZH.png)

* **腳本有帶傳入參數**

其中 `2` 代表前兩個傳入參數將帶入 KEYS，而後續參數則會依序帶入 ARGV

![](https://i.imgur.com/4Jbx52O.png)

#### SCRIPT LOAD `<script>`

將腳本存於內存且不會立即執行，若成功則回傳該腳本的 SHA1 雜湊值，若已存在則忽略不儲存

![](https://i.imgur.com/gPG4LfA.png)

#### EVALSHA `<hash>`

執行內存中已存在的 SHA1 雜湊，並執行對應的腳本

![](https://i.imgur.com/CfT7PVM.png)


#### SCRIPT EXISTS `<hash1> <hash2> ...`

可判斷多個內存腳本是否存在，存在返回 1, 不存在返回 0

![](https://i.imgur.com/05XKz3P.png)

#### SCRIPT FLUSH

清除 server 內存所有腳本

![](https://i.imgur.com/Y1RzzX3.png)



#### SCRIPT KILL

砍掉 server 當前運行的腳本，若無腳本運行則回 `NOTBUSY ...` 錯誤訊息

![](https://i.imgur.com/LmLDXkz.png)