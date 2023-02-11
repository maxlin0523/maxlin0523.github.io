---
title: ASP.NET Core Distributed Cache
catalog: true
date: 2023-02-11 22:13:13
subtitle:
header-img:
tags:
- AspNetCore
- Cache
---
如果說系統架構是多台的情況呢？例如：Load Balance，這時就適合用 `分散式快取` 。

簡單來說，`分散式快取` 就是統一將快取資料存在某個 `儲存體`，當多台機器去查詢資料時，就會指向該 `儲存體` 來進行存取。

這邊選用 Redis 來當作分散式快取的 `儲存體` 。

### 在實作前

1. 先將 Redis 架在 `localhost:6379` (架設可[參考](https://maxlin0523.github.io/2021/10/09/redis-in-docker/))，並將連線字串加入 appsettings.json
2. 將專案參考 `Microsoft.Extensions.Caching.StackExchangeRedis`


### 於 Program.cs 注入 IDistributedCache

如下：
```C#=
// 取得 appsettings 中的 Redis 連線字串
var redisConStr = builder.Configuration.GetSection("RedisCache:ConnectionString").Value;
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = redisConStr;
});
```

### 在建構式傳入 IDistributedCache 實體

如下：
```c#=
private readonly IDistributedCache _distributedCache;

public DistributedCacheController(IDistributedCache distributedCache)
{
    _distributedCache = distributedCache;
}
```

### 取得快取資料

如下：
```C#=
[HttpGet("{userId}")]
public async Task<ActionResult<UserInfo>> Get(int userId)
{
    var source = await _distributedCache.GetAsync(userId.ToString());
    if (source == null)
    {
        return NotFound();         
    }
    
    var info = JsonSerializer.Deserialize<UserInfo>(source);
    return info;
}
```

### 更新快取資料

如下：
```C#=
[HttpPost]
public async Task<ActionResult> Set([FromBody] UserInfo info)
{
    await _distributedCache.SetStringAsync(info.Id.ToString(), JsonSerializer.Serialize(info));
    return NoContent();
}
```

### 刪除快取資料

如下：
```C#=
[HttpDelete("{userId}")]
public async Task<ActionResult> Remove(int userId)
{
    await _distributedCache.RemoveAsync(userId.ToString());
    return NoContent();
}
```


#### 取得或新增快取資料

因為已習慣使用 `MemoryCache` 的 GetOrCreate，所以我也刻了一個給 `IDistributedCache` 來使用。

然後快取時間就依情境設定即可
```c#=
[HttpGet("{userId}")]
public async Task<ActionResult<UserInfo>> GetOrCreate(int userId)
{
    var cacheEntryOptions = new DistributedCacheEntryOptions
    {
        // Set AbsoluteExpiration for 3 mins
        AbsoluteExpiration = DateTimeOffset.Now.AddMinutes(3),
        // Set SlidingExpiration for 5 s
        SlidingExpiration = TimeSpan.FromSeconds(5)
    };

    var info = await _distributedCache.GetOrCreateAsync(userId.ToString(), () => GetUserInfoFromDbAsync(), cacheEntryOptions);
    return Ok(info);
}
```

擴充方法代碼如下：

```c#=
public static async Task<TItem> GetOrCreateAsync<TItem>(this IDistributedCache cache, string key, Func<Task<TItem>> entity, DistributedCacheEntryOptions options = null)
{
    if (options == null)
    {
        options = new DistributedCacheEntryOptions()
        {
            // 預設 5 分鐘快取
            AbsoluteExpiration = DateTimeOffset.Now.AddMinutes(5)
        };
    }

    var value = await cache.GetStringAsync(key);
    if (value == null)
    {
        var source = await entity();
        var jsonSource = JsonSerializer.Serialize(source);
        await cache.SetStringAsync(key, jsonSource, options);
        return source;
    }
    return JsonSerializer.Deserialize<TItem>(value);
}
```

## 參考：

[範例原始碼](https://github.com/maxlin0523/aspnetcore.caching.demo)

[ASP.NET Core中的分散式快取](https://learn.microsoft.com/zh-tw/aspnet/core/performance/caching/distributed?view=aspnetcore-7.0)


