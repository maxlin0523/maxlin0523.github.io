---
title: C# 使用 Redis Script
catalog: true
date: 2022-07-25 01:48:11
subtitle:
header-img:
tags:
- Redis
- Lua
- C#
---
先前大致有紀錄 [基礎 LuaScript 語法](/2021/10/16/luascript/)、[Docker 佈署 Redis](/2021/10/09/redis-in-docker/) 以及 [Redis 腳本](/2021/12/11/redis-lua-script/)，接下來紀錄一下 C# 與 Redis 腳本串接方式。

連線：

```c#=
var connString = "localhost:6379";

// 建立連線
ConnectionMultiplexer conn = ConnectionMultiplexer.Connect(connString);

// 取得 redis server 實體
IServer redisServer = conn.GetServer(connString);
```

建立資料：

```c#=
var createScript = @"
local key = KEYS[1]
local id = ARGV[1]
local name = ARGV[2]
local phone = ARGV[3]
return redis.call('HSET', key, 'Id', id, 'Name', name, 'Phone', phone)"; // do redis command

// 載入至 server
var loadedScript = await LuaScript.Prepare(createScript).LoadAsync(redisServer);

// 宣告入參
var keys = new RedisKey[] { "Profile:Max" };
var values = new RedisValue[] { "A128996644", "Max Lin", "3345678" };

// 執行腳本
await conn.GetDatabase().ScriptEvaluateAsync(loadedScript.Hash, keys, values);
```

查看：

![](https://i.imgur.com/LzT298z.png)

取得資料：

```C#=
var getScript = @"
local key = KEYS[1]
return redis.call('HVALS', key)";

// 載入至 server
var loadedScript = await LuaScript.Prepare(getScript).LoadAsync(redisServer);

// 執行腳本
var result = (string[])await conn.GetDatabase().ScriptEvaluateAsync(loadedScript.Hash, new RedisKey[] { "Profile:Max" });

var profile = result.Length == 0 ? null : new Profile
{
    Id = result[0],
    Name = result[1],
    Phone = int.Parse(result[2])
};

Console.WriteLine("Profile： ");
Console.WriteLine(profile.Id);
Console.WriteLine(profile.Name);
Console.WriteLine(profile.Phone);
Console.ReadKey();

```

![](https://i.imgur.com/3VlkWhP.png)

## 補充

在使用 lua script 執行 redis command 時，分別有兩種方式：
* redis.call
* redis.pcall

兩者差別主要在於錯誤處理方式，在調用 `redis.call` 時出現錯誤，則會將錯誤向上拋出，而 `redis.pcall` 則會將錯誤 catch 並回傳一個錯誤訊息。