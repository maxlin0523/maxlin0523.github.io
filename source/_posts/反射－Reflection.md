---
title: C# Reflection
catalog: true
date: 2021-06-21 00:05:23
subtitle:
header-img:
tags: 
- C# 本事
---
# 介紹
反射 (Reflection) 會提供 `Type` 型別的物件，用來封裝組件、模組和型別。您可以使用反射來動態建立型別的執行個體、將型別繫結至現有物件，或從現有物件取得型別，並叫用其方法或存取其欄位和屬性（[出自MSDN](https://docs.microsoft.com/zh-tw/previous-versions/ms173183(v=vs.80)?redirectedfrom=MSDN)）


簡單來說

只要把類別物件轉換成 `Type` 型別，藉由 `Type` 型別，可以反射出類別物件本身或其內部成員的各種資訊。例如：方法、屬性、欄位、建構式、資料型態、存取修飾詞（如 `public`/`private` ）．．．等，如果是透過實體物件轉換的情況，實體物件成員的賦值也是可以取得的。
總之，在這個物件能呼叫到的成員大都可以透過反射取得，甚至無法直接呼叫的私有成員也可以。

可以看到 `Type` 提供的屬性數量（多到爆XD），若要反射出那麼多的資訊，勢必會犧牲掉效能，就好比你需要查某個生字的單詞造句，所以需要利用一本字典查詢，但那本字典同時也建了所有生字的單詞造句供你使用。

效能問題可[參考](https://www.gushiciku.cn/pl/2s9p/zh-tw)

# 基本範例

###### 定義一個要測試的類別，有公開屬性、公開方法、私有方法
![](https://i.imgur.com/ks8XFUa.png)

###### 反射類別內所有公開屬性
![](https://i.imgur.com/ayhF09Z.png)


###### 反射類別內所有公開方法
![](https://i.imgur.com/TDlvjzh.png)

###### 驗證定義的公開成員和反射是否一樣
![](https://i.imgur.com/qFw2AMJ.png)


確實都一樣，有一些是屬於內建的公開成員！

如果想反射非公開方法呢？
那會需要加入 `Bindflags` 列舉型別參數告訴 `GetMethods` 要搜尋哪些存取範圍
這邊設定為實體 `Instance`＋非公開 `NonPublic`
![](https://i.imgur.com/U18dg0c.png)

如此一來就能輕鬆反射出非公開方法！
當然，想要哪些存取範圍都可以自定義 Bindflags 參數去搜尋，詳細請[參考](https://blog.csdn.net/qq_32452623/article/details/53401890)

# 應用範例(使用SQL Server)
當我們使用 Dapper 做 CRUD 時，常常需要組裝 SQL 語法參數，而當要做的資料表或資料表欄位很多時，就會覺得很麻煩要一個個打上去，這時候就是使用反射的時機
直接開始實作！

###### 定義要測試的資料表
###### 名稱：NBA
###### 欄位：PlayerName、PlayerNumber、LeagueTeam、Position
![](https://i.imgur.com/VXNzu1V.png)



###### 我們可以定義一個類別與資料表名稱欄位一致
![](https://i.imgur.com/fPyuiFy.png)

若不想名稱一致可在後續範例方法自行新增參數去調整，或不想欄位一致也可自建 Dictionary 匹配。
有了類別接下來就可以利用反射慢慢把語法拼湊出來。
###### 建立（ SqlCommand.cs ）
```C#
/// <summary>
/// 根據反射類別，映射出SQL語法。
/// ※資料表名稱預設為類別名
/// </summary>
public class SqlCommand
{

}
```

再依序加入

###### Insert
```C#=
/// <summary>
/// Insert語法組裝
/// </summary>
public static string Insert<T>() where T : class
{
    var type = typeof(T)
    var props = type.GetProperties()
    // 取得欄位加入內存
    var columns = props.Select(prop =>
        prop.Name)
    var values = props.Select(prop =>
        $"@{prop.Name}")
    // 欄位
    var columnString = string.Join(",\r\n", columns)
    // 欄位參數
    var valueString = string.Join(",\r\n", values)
    var table = type.Name
    // 組裝
    var sql = new StringBuilder();
    sql.AppendLine($"INSERT INTO dbo.{table}");
    sql.AppendLine($"(");
    sql.AppendLine(columnString);
    sql.AppendLine($")");
    sql.AppendLine($"VALUES");
    sql.AppendLine($"(");
    sql.AppendLine(valueString);
    sql.AppendLine($")");
    return sql.ToString();
}
```

###### Select
```C#=
/// <summary>
/// Select語法組裝
/// </summary>
public static string Select<T>(object param = null) 
    where T : class
{
    var type = typeof(T)
    // 取得欄位並加入內存
    var columns = type.GetProperties()
        .Select(prop => prop.Name)
    // 條件式內存
    var conditions = new List<string>()
    // 檢查有無條件式參數
    if (param != null)
    {
        // 取得條件式並組裝加入條件式內存
        conditions = param.GetType()
            .GetProperties()
            .Select(prop => $"{prop.Name} = @{prop.Name}")
            .ToList()
        // 條件式前加上WHERE
        conditions[0] = $"WHERE {conditions[0]}";
    
    var columnString = string.Join(",\r\n", columns)
    var conditionString = string.Join("\r\nAND ", conditions)
    var table = type.Name
    // 組裝
    var sql = new StringBuilder();
    sql.AppendLine("SELECT");
    sql.AppendLine(columnString);
    sql.AppendLine($"FROM dbo.{table}");
    sql.AppendLine(conditionString);
    return sql.ToString();
}
```
###### Update
```C#=
/// <summary>
/// Update語法組裝
/// </summary>
public static string Update<T>(object param = null) 
    where T : class
{
    var type = typeof(T)
    // 取得欄位並組裝加入內存
    var columns = type.GetProperties()
        .Select(prop => $"{prop.Name} = @{prop.Name}")
        .ToList()
    // 條件式內存
    var conditions = new List<string>()
    // 檢查有無條件式
    if (param != null)
    {
        // 取得條件式並組裝加入條件式內存
        conditions = param.GetType()
            .GetProperties()
            .Select(prop => $"{prop.Name} = @{prop.Name}")
            .ToList()
        // 條件式參數不需更新，故排除
        columns = columns.Except(conditions).ToList()
        // 條件式前面加上WHERE
        conditions[0] = $"WHERE {conditions[0]}";
    
    // 欄位
    var columnString = string.Join(",\r\n", columns)
    // 條件式
    var conditionString = string.Join("\r\nAND ", conditions)
    var table = type.Name
    // 組裝
    var sql = new StringBuilder();
    sql.AppendLine($"UPDATE dbo.{table}");
    sql.AppendLine($"SET");
    sql.AppendLine(columnString);
    sql.AppendLine(conditionString);
    return sql.ToString();
}
```

###### Delete
```C#=
/// <summary>
/// Delete語法組裝
/// </summary>
public static string Delete<T>(object param = null) 
    where T : class
{
    var type = typeof(T);

    // 條件式內存
    var conditions = new List<string>();

    // 判斷有無條件式參數
    if (param != null)
    {
        // 取得條件式並組裝加入條件式內存
        conditions = param.GetType()
            .GetProperties()
            .Select(prop => $"{prop.Name} = @{prop.Name}")
            .ToList();

        // 條件式前加上WHERE
        conditions[0] = "WHERE " + conditions[0];
    }

    var conditionString = string.Join("\r\nAND ", conditions);

    var table = type.Name;

    // 組裝
    var sql = new StringBuilder();
    sql.AppendLine($"DELETE FROM dbo.{table}");
    sql.AppendLine(conditionString);
    return sql.ToString();
}
```


這樣就完成了！
可能有人疑惑為什麼一堆換行(`\r\n`)、`AppendLine` 之類的 code？
目的是排版，當效能不允許使用時反射時，也可直接用範例 `return` 出的字串來微調使用。
###### 測試
```C#=
  
using ReflectionSample.Models;
using System;

namespace ReflectionSample
{
    class Program
    {
        static void Main(string[] args)
        {
            var param = new NBA();

            Console.WriteLine("新增");
            var post = SqlCommand.Insert<NBA>();
            Console.WriteLine(post);


            Console.WriteLine("查詢");
            var get = SqlCommand.Select<NBA>(
                new { param.LeagueTeam});
            Console.WriteLine(get);

            Console.WriteLine("修改");
            var put = SqlCommand.Update<NBA>(
                new { param.PlayerName, param.PlayerNumber });
            Console.WriteLine(put);


            Console.WriteLine("刪除");
            var delete = SqlCommand.Delete<NBA>(
                new { param.PlayerName });
            Console.WriteLine(delete);


            Console.ReadKey();
        }
    }
}
```

輸出
```
新增
INSERT INTO dbo.NBA
(
PlayerName,
PlayerNumber,
LeagueTeam,
Position
)
VALUES
(
@PlayerName,
@PlayerNumber,
@LeagueTeam,
@Position
)

查詢
SELECT
PlayerName,
PlayerNumber,
LeagueTeam,
Position
FROM dbo.NBA
WHERE LeagueTeam = @LeagueTeam

修改
UPDATE dbo.NBA
SET
LeagueTeam = @LeagueTeam,
Position = @Position
WHERE PlayerName = @PlayerName
AND PlayerNumber = @PlayerNumber

刪除
DELETE FROM dbo.NBA
WHERE PlayerName = @PlayerName
```

範例原始碼：https://github.com/maxlin0523/ReflectionSample

還可參考另一篇[分層架構](https://maxlin0523.github.io/2021/05/16/%E5%88%86%E5%B1%A4%E6%9E%B6%E6%A7%8B%EF%BC%8DLayered%20Architecture/)， `Repository` 層就是應用 `Dapper` + `Reflection` 。
# 參考
[C# 反射（Reflection）](https://www.runoob.com/csharp/csharp-reflection.html)
