---
title: ASP.NET Core Graceful Shutdown
catalog: true
date: 2023-11-22 00:03:50
subtitle:
header-img:
tags:
- AspNetCore
---
目的是當 application 結束時, 能夠優雅的使其關閉，像是留有足夠的緩衝時間來消化 application 內執行中的 task 以減少不必要的損害。



## 範例

假設目前需要將 Application 關閉重啟，但是 `Task1` 和 `Task2` 正再執行，且分別需要 10 秒和 5 秒，因此我們可以將延遲關閉時間訂一個合適的時間，範例為 20 秒，

如：

```C#=
WebApplication app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.Lifetime.ApplicationStopping.Register(() => {
    app.Logger.LogInformation($"{DateTime.Now:mm:ss.fff} ApplicationStopping");
    Task.Run(async () => await Task1());  // Task1 excuting
    Task.Run(async () => await Task2());  // Task2 excuting
    Task.Delay(20000).Wait(); // Wait for 20 s, then it's done."
});


app.Lifetime.ApplicationStopped.Register(() => {
    app.Logger.LogInformation($"{DateTime.Now:mm:ss.fff} ApplicationStopped");
});

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();

async Task Task1() 
{
    app.Logger.LogInformation($"{DateTime.Now:mm:ss.fff} Task1 has not been completed.");
    await Task.Delay(10000);
    app.Logger.LogInformation($"{DateTime.Now:mm:ss.fff} Task1 is completed.");
}

async Task Task2()
{
    app.Logger.LogInformation($"{DateTime.Now:mm:ss.fff} Task2 has not been completed.");
    await Task.Delay(5000);
    app.Logger.LogInformation($"{DateTime.Now:mm:ss.fff} Task2 is completed.");
}
```

![](../dotnet-graceful-shutdown/1.png)

## 參考


https://github.com/maxlin0523/graceful-shutdown

[ASP.NET Core 停機行為觀察](https://blog.darkthread.net/blog/aspnet-core-shutdown/)
