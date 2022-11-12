---
title: C# 使用 CsvHelper 套件讀取 Csv 檔
catalog: true
date: 2022-04-09 18:22:58
subtitle:
header-img:
tags: 
- Nuget
- C#
---
紀錄 Nuget 套件 `CsvHelper` 幾個不同情況的 `讀取` 方式。



## 1. 當 csv 欄位與類別名稱與資料型態可對應

資料格式：

| Id  | Name  | Email           | Phone      |
| --- | ----- | --------------- | ---------- |
| 0   | Eric  | Eric@gmail.com  | 0911111111 |
| 1   | Amy   | Amy@gmail.com   | 0922222222 |
| 2   | Titan | Titan@gmail.com | 0933333333 |

資料源：
```
Id,Name,Email,Phone
0,Eric,Eric@gmail.com,0911111111
1,Amy,Amy@gmail.com,0922222222 
2,Titan,Titan@gmail.com,0933333333
```

做法如下：

```C#
static void Main(string[] args)
{
    // Csv 配置
    CsvConfiguration setting = new CsvConfiguration(CultureInfo.InvariantCulture);

    using (StreamReader sr = new StreamReader("./data.csv"))
    using (CsvReader csv = new CsvReader(sr, setting))
    {
        // 取得資料
        List<User> result = csv.GetRecords<User>().ToList();
    }

}
public class User
{
    public string Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }
}

```

##  2. 讀取不含 csv 標題

資料格式：
| 0   | Eric  | Eric@gmail.com  | 0911111111 |
| --- | ----- | --------------- | ---------- |
| 1   | Amy   | Amy@gmail.com   | 0922222222 |
| 2   | Titan | Titan@gmail.com | 0933333333 |

資料源：
```
0,Eric,Eric@gmail.com,0911111111
1,Amy,Amy@gmail.com,0922222222 
2,Titan,Titan@gmail.com,0933333333
```


需調整 **Configuration**，做法如下：

```C#
static void Main(string[] args)
{
    // Csv 配置
    CsvConfiguration setting = new CsvConfiguration(CultureInfo.InvariantCulture)
    {
    　　 // 不含標題
        HasHeaderRecord = false
    };

    using (StreamReader sr = new StreamReader("./data.csv"))
    using (CsvReader csv = new CsvReader(sr, setting))
    {
        // 取得資料
        List<User> result = csv.GetRecords<User>().ToList();
    }

}
public class User
{
    public string Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }
}

```

## 3. 當 csv 欄位名與類別屬性無法對應

資料格式：
| 編號  | 名稱  | 信箱           | 手機      |
| --- | ----- | --------------- | ---------- |
| 0   | Eric  | Eric@gmail.com  | 0911111111 |
| 1   | Amy   | Amy@gmail.com   | 0922222222 |
| 2   | Titan | Titan@gmail.com | 0933333333 |

資料源：
```
編號,名稱,信箱,手機
0,Eric,Eric@gmail.com,0911111111
1,Amy,Amy@gmail.com,0922222222 
2,Titan,Titan@gmail.com,0933333333
```

可建立 **ClassMap** 去定義對應情境，完整做法如下：

```c#
static void Main(string[] args)
{
    // Csv 配置
    CsvConfiguration setting = new CsvConfiguration(CultureInfo.InvariantCulture);

    using (StreamReader sr = new StreamReader("./data.csv"))
    using (CsvReader csv = new CsvReader(sr, setting))
    {
        // 註冊
        csv.Context.RegisterClassMap<UserMap>();
        
        // 取得資料
        List<User> result = csv.GetRecords<User>().ToList();
    }

}
public class User
{
    public string Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }
}

public sealed class UserMap : ClassMap<User>
{
    public UserMap()
    {
        // 屬性對應欄位名稱
        Map(m => m.Id).Name("編號");
        Map(m => m.Name).Name("名稱");
        Map(m => m.Email).Name("信箱");
        Map(m => m.Phone).Name("手機");
    }
}
```

或是用類別屬性去對應 csv 欄位索引，如下：

```c#
public sealed class UserMap : ClassMap<User>
{
    public UserMap()
    {
        // 屬性對應欄位索引
        Map(m => m.Id).Index(0);
        Map(m => m.Name).Index(1);
        Map(m => m.Email).Index(2);
        Map(m => m.Phone).Index(3);
    }
}
```


## 4. 當 csv 欄位名稱有模稜兩可的情況

例如：讀檔時，第一個欄位名稱可能出現`編號` or `號碼` 的情形，如下：

```
編號,名稱,信箱,手機
0,Eric,Eric@gmail.com,0911111111

or 

號碼,名稱,信箱,手機
0,Eric,Eric@gmail.com,0911111111
```

則 **ClassMap** 可改為：

```C#
public sealed class UserMap : ClassMap<User>
{
    public UserMap()
    {
        // 屬性對應欄位名稱
        Map(m => m.Id).Name("編號", "號碼");
        Map(m => m.Name).Name("名稱");
        Map(m => m.Email).Name("信箱");
        Map(m => m.Phone).Name("手機");
    }
}
```

## 5. 忽略類別屬性對應

資料格式：

| Id  | Name  | Email           | Phone      |
| --- | ----- | --------------- | ---------- |
| 0   | Eric  | Eric@gmail.com  | 0911111111 |
| 1   | Amy   | Amy@gmail.com   | 0922222222 |
| 2   | Titan | Titan@gmail.com | 0933333333 |

資料源：
```
Id,Name,Email,Phone
0,Eric,Eric@gmail.com,0911111111
1,Amy,Amy@gmail.com,0922222222 
2,Titan,Titan@gmail.com,0933333333
```

當類別多出 `Description` 屬性，做法如 **ClassMap**：

```C#
static void Main(string[] args)
{
    // Csv 配置
    CsvConfiguration setting = new CsvConfiguration(CultureInfo.InvariantCulture);

    using (StreamReader sr = new StreamReader("./data.csv"))
    using (CsvReader csv = new CsvReader(sr, setting))
    {
        // 取得資料
        csv.Context.RegisterClassMap<UserMap>();
        List<User> result = csv.GetRecords<User>().ToList();
    }

}
public class User
{
    public string Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }

    public string Description { get; set; }
}

public sealed class UserMap : ClassMap<User>
{
    public UserMap()
    {
        Map(m => m.Id);
        Map(m => m.Name);
        Map(m => m.Email);
        Map(m => m.Phone);
        Map(m => m.Description).Ignore();
    }
}
```

## 6. 當 csv 欄位有 Json 格式資料

資料格式：



| ClassId | Classmate | 
| -------- | -------- | 
| 3     | [""Andy"",""Eric"",""Max""]     | 


資料源：
```
ClassId,Classmate
3,"[""Andy"",""Eric"",""Max""]"

```

**ClassMap** 對應做法如下：

```c#
static void Main(string[] args)
{
    // Csv 配置
    CsvConfiguration setting = new CsvConfiguration(CultureInfo.InvariantCulture);

    using (StreamReader sr = new StreamReader("./data.csv"))
    using (CsvReader csv = new CsvReader(sr, setting))
    {
        // 取得資料
        csv.Context.RegisterClassMap<ClassroomMap>();
        List<Classroom> result = csv.GetRecords<Classroom>().ToList();
    }

}
public class Classroom
{
    public string ClassId { get; set; }

    public List<string> Classmate { get; set; }
}

public sealed class ClassroomMap : ClassMap<Classroom>
{
    public ClassroomMap()
    {
        Map(m => m.ClassId);
        Map(m => m.Classmate).Convert(row => JsonSerializer.Deserialize<List<string>>(row.Row.GetField("Classmate")));
    }
}
```





## 參考
[[C#][.NET Core] CsvHelper : 透過 C# 讀寫 csv 檔案](https://dog0416.blogspot.com/2019/11/aspnet-core-csvhelper-c-csv.html?fbclid=IwAR2HnqnMGJhYcjnMBrcBKBYiR2JyIAkCA_NZlMPbrUiUnmaII9MpSWgquqk)

[使用 CsvHelper - Part.2 資料讀取](https://kevintsengtw.blogspot.com/2015/04/csvhelper-part2.html)

[【C#】CsvHelper 使用手册](https://www.cnblogs.com/gl1573/p/12922857.html)

[CsvHelper 官方文檔](https://joshclose.github.io/CsvHelper/examples/)