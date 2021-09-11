---
title: .NET Core 之效能監控 CoreProfiler
catalog: true
date: 2021-08-11 21:31:51
subtitle:
header-img:
tags: 
- Nuget 套件
- 效能監控
---
# 介紹
`CoreProfiler` 是一個基於 `.NET Core` 框架開發的效能監控工具，而使用方式非常簡單，把專案裡需要監控的範圍用 `using` 陳述式包起來就可以了，而當 `using` 範圍結束就會自動幫我們紀錄耗時，大概使用方式就這樣！簡單易用好懂！


# 實作( .NET Core WebApi )
<br>

### 環境建置
* 安裝 `CoreProfiler`、`CoreProfiler.Web` 套件

* 建立 `coreprofiler.json`，輸出目錄：**永遠複製**
![](https://i.imgur.com/oR8JsmO.png)


* 把設定配置加入至 `coreprofiler.json`
```json=
{
  "circularBufferSize": 200,
  "filters": [
    {
      "key": "/coreprofiler",
      "value": "/coreprofiler",
      "type": "CoreProfiler.ProfilingFilters.NameContainsProfilingFilter, CoreProfiler"
    },
    {
      "key": "static files",
      "value": "ico,jpg,js,css,svg,json,ttf,woff,woff2,eot",
      "type": "CoreProfiler.ProfilingFilters.FileExtensionProfilingFilter, CoreProfiler"
    }
  ]
}
```


* 在 `Startup.Configure` 添加 `app.UseCoreProfiler(true);`
<br>

### 基礎用法


* 在預設的 `WeatherForecastController.Get()` 方法裡加入監控，另外在設定延遲5秒方便觀察
```C#=
[HttpGet]
public IEnumerable<WeatherForecast> Get()
{
            //　監控名稱
            var stepName = $"監控名稱";
            using (ProfilingSession.Current.Step(stepName))
            {
                var rng = new Random();
                // 延遲5秒鐘
                Thread.Sleep(5000);
                return Enumerable.Range(1, 5).Select(index => new WeatherForecast
                {
                    Date = DateTime.Now.AddDays(index),
                    TemperatureC = rng.Next(-20, 55),
                    Summary = Summaries[rng.Next(Summaries.Length)]
                })
                .ToArray();
            }
}
```
* 專案跑起來後，先打入要監控的 API：`httpGet` => `https://<域名>/WeatherForecast`

* 將域名後加入`/coreprofiler/view` 進入監控頁面 ，Ex： `https://<域名>/coreprofiler/view`

* 點選此次 Request API 的詳細內容

![](https://i.imgur.com/dWSXjvR.png)

* 查看

![](https://i.imgur.com/Cqws7ky.png)

done.
<br>

### ActionFilter 
我們都知道 `using` 陳述式中的參數必須是 `IDisposable` 型別，當 `using` 範圍結束後會自動 Dispose，若不用 `using` 的話，就要在 `IDisposable` 型別參數使用完後加上 Dispose 方法，才會釋放掉記憶體。

所以，我們可以用 ActionFilter 的方式在 API 生命週期的開始時監控，結束時在手動寫下 Dispose 紀錄耗時，如下：
```C#=
public class CoreProfilingAttribute : ActionFilterAttribute
{
    /// <summary>
    /// ProfilingName.
    /// </summary>
    public string ProfilingName { get; set; }

    /// <summary>
    /// ProfilingStep.
    /// </summary>
    public IDisposable ProfilingStep { get; set; }

    /// <summary>
    /// 執行前
    /// </summary>
    public override void OnActionExecuting(ActionExecutingContext context)
        {
            base.OnActionExecuting(context);

            if (string.IsNullOrEmpty(this.ProfilingName))
            {
                this.ProfilingName = context.ActionDescriptor.DisplayName;
            }

            // 開始監控
            this.ProfilingStep = ProfilingSession.Current.Step(this.ProfilingName);
        }

    /// <summary>
    /// 執行後
    /// </summary>
    public override void OnActionExecuted(ActionExecutedContext context)
    {
        base.OnActionExecuted(context);
        
        // 監控結束 Dispose
        this.ProfilingStep?.Dispose();
    }
}
```

掛載到 Function 上

![](https://i.imgur.com/EjqNFeO.png)

done.
<br>

### IDbConnection 監控
剛介紹了 API 的生命週期監控，若還想額外監控 API 在 Db Connection 的耗時呢？如下：

IDbConnection 改為
```C#=
var connection = new ProfiledDbConnection
(
    new SqlConnection(this.ConnectionString),
    () => ProfilingSession.Current == null
        ? null
        : new DbProfiler(ProfilingSession.Current.Profiler)
);
```

用剛的 API 改寫
```C#=
[HttpGet]
[CoreProfiling]
public async Task<IEnumerable<int>> Get()
{
    // delay 500 ms
    Thread.Sleep(500);
    
    using (var conn = GetConnection())
    {
        var sql = @"SELECT Id FROM dbo.Sample";
        
        // Dapper
        var result = await conn.QueryAsync<int>(sql);
        return result;
    }
}

private IDbConnection GetConnection()
{
    var connectionString = @"<Your ConnectionString>";

    var connection = new ProfiledDbConnection
    (
        new SqlConnection(connectionString),
        () => ProfilingSession.Current == null
            ? null
            : new DbProfiler(ProfilingSession.Current.Profiler)
    );

    return connection;
}
```

查看監控頁面
![](https://i.imgur.com/tTyMfYs.png)

done.
<br>

# 參考
[套件 SourceCode](https://github.com/teddymacn/CoreProfiler)
[monitor performance of webapi via CoreProfiler](https://dotblogs.com.tw/ricochen/2018/04/02/023518?fbclid=IwAR3FCRqZqDoqw-yDTMgU_2mJEMD1031JJlnQfjIWrXKKkukB3diIMIQ2eOU)

