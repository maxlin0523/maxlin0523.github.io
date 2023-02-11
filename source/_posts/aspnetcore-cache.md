---
title: ASP.NET Core Cache
catalog: true
date: 2023-02-11 22:00:22
subtitle:
header-img:
tags: 
- AspNetCore
- Cache
---
快取主要是將資料暫存於記憶體，方面日後調用時能有更好的回應速度以及重用性，同時也減少對資源的消耗(如：網路、Disk I/O)。

舉一個適用場景：

當查詢的資料 `不常變更` 時，如：`歷史資料`，其多次查詢大多結果都相同，這時就很適合把該資料存進快取，當查詢時就直接從快取獲得資料並回傳給使用方而不必透過資料庫存取，也印證了開頭提到的特點。

### 於 Program.cs 注入 MemoryCache

如下：

```C#=
builder.Services.AddMemoryCache();
```

### 在建構式傳入 IMemoryCache 實體

如下：

```c#=
private readonly IMemoryCache _memoryCache;

public MemoryCacheController(IMemoryCache memoryCache)
{
    _memoryCache = memoryCache;
}
```

### 新增快取資料

※ 如新增重複 key 會直接覆蓋舊 value

```C#=
[HttpPost]
public ActionResult Set([FromBody] UserInfo info)
{
    // 依 id 儲存資料於快取
    _memoryCache.Set(info.Id, info);
    return NoContent();
}
```

### 刪除快取資料

※ 如刪除不存在 key 不會出現例外

```C#=
[HttpDelete("{userId}")]
public ActionResult Remove(int userId)
{
    // 依 id 刪除快取資料
    _memoryCache.Remove(userId);
    return NoContent();
}
```


### 取得快取資料

分幾種方式：

第一種(單純 `get`)：

```C#=
var info = _memoryCache.Get<UserInfo>(userId);
if (info == null)
{
    // 找不到的情況
    throw KeyNotFoundException();
}

return info;
```

第二種(`TryGetValue`)：

```C#=
if (_memoryCache.TryGetValue<UserInfo>(userId, out var info) == false)
{
    // 找不到的情況
    throw KeyNotFoundException();
}
return info;
```

第三種(`GetOrCreate` 個人較常用)：

此方式支援同步(`GetOrCreate`)和非同步(`GetOrCreateAsync`)

```C#=
// 找到就直接回傳，找不到就去 GetUserInfoFromDb 取
var info = _memoryCache.GetOrCreate(userId, (entry) =>
{
    return GetUserInfoFromDb();
});

return info;
```

### 設定快取過期時間(TTL)

過期機制分為：

* Absolute：設定的過期時間到了將直接刪除
* Sliding：在設定的過期時間內，如沒有存取將會 `刪除`，若有存取則會 `重置` 過期時間

```c#=
var info = _memoryCache.GetOrCreate(userId, (entry) =>
{
    // Set AbsoluteExpiration for 1 day
    entry.AbsoluteExpiration = DateTimeOffset.Now.AddDays(1);
    // Set SlidingExpiration for 1 hour
    entry.SlidingExpiration =  TimeSpan.FromHours(1);

    return GetUserInfoFromDb();
});

return info;
```

## 參考：

[範例原始碼](https://github.com/maxlin0523/aspnetcore.caching.demo)
