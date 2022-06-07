---
title: C# 串接 MongoDB
catalog: true
date: 2022-05-22 02:42:47
subtitle:
header-img:
tags: 
- MongoDB
---
### 前言
雖說之前有看過 [Mongo University](https://university.mongodb.com/courses/catalog)，也了解過一些 MongoDB 相關知識，但許久沒用都忘的差不多了，所以重新學習並記錄一下使用方式。


### 環境建置(使用 Docker) 

Pull Image：
`docker pull mongo`

Run in Container：
`docker run --name samplemongo -v ./data:/data/db -d -p 27017:27017 --rm mongo:latest`

![](https://i.imgur.com/MlTR8r1.png)

下載 GUI ( `MongoDB Compass` ) 並進行連線：
![](https://i.imgur.com/0Hd4PeD.png)

成功後，回到專案內下載 Nuget( `MongoDB.Driver` )：
![](https://i.imgur.com/YB3rgr8.png)

### 使用方式

定義好要存的資料型態：

```c#
public class CourseModel
{
    [BsonId]
    public string CourseId { get; set;}

    [BsonElement]
    public string CourseName { get; set; }

    [BsonElement]
    public string Teacher { get; set; }

    [BsonElement]
    public List<Student> Students { get; set; }
}

public class Student
{
    public int Id { get; set; }

    public string Name { get; set; }
}
```

接著建立連線並寫入一筆資料：

```C#
static async Task Main(string[] args)
{
    // 連線字串
    string connectionString = "mongodb://localhost:27017";
    // 連接 MongoDB
    MongoClient client = new MongoClient(connectionString);
    // 取得 MongoDB 資料庫
    IMongoDatabase database = client.GetDatabase("Sample");
    // 取得 Collection(相當於 sql 的資料表)
    IMongoCollection<CourseModel> collection = database.GetCollection<CourseModel>("Course");
    
    CourseModel course = new CourseModel
    {
        CourseId = "AA123",
        CourseName = "英文",
        Teacher = "Amy",
        Students = new List<Student>
        {
            new Student { Id = 1314, Name = "Feii" },
            new Student { Id = 1315, Name = "Danny" }
        }
    };

    // 寫入至 MongoDB
    await collection.InsertOneAsync(course);
}
```

查看寫入資料：

![](https://i.imgur.com/XwRN0pT.png)


### Mongo CRUD

這邊我先建立一個類別存取配置檔

```C#=
public class CourseMongoDatabaseSetting
{
    /// <summary>
    /// 連線字串
    /// </summary>
    public string ConnectionString { get; set; }
    /// <summary>
    /// 資料庫
    /// </summary>
    public string Database { get; set; }
    /// <summary>
    /// Collection
    /// </summary>
    public string Collection { get; set; }
}
```

接著建立 `MongoRepository` 建構子吃 `CourseMongoDatabaseSetting` 類別

```c#
public class MongoRepository
{
    private readonly IMongoCollection<CourseModel> _collection;

    public MongoRepository(CourseMongoDatabaseSetting setting)
    {
        MongoClient client = new MongoClient(setting.ConnectionString);

        IMongoDatabase database = client.GetDatabase(setting.Database);

        _collection = database.GetCollection<CourseModel>(setting.Collection);
    }

    /// <summary>
    /// 建立課程
    /// </summary>
    public async Task<bool> CreateCourse(CourseModel model)
    {
        await _collection.InsertOneAsync(model);
        return true;
    }

    /// <summary>
    /// 更新課程
    /// </summary>
    public async Task<bool> UpdateCourse(UpdateCourseModel model)
    {
        var update = new UpdateDefinitionBuilder<CourseModel>()
            .Set(x => x.Teacher, model.Teacher)
            .Set(x => x.CourseName, model.CourseName);

        await _collection.UpdateOneAsync(x => x.CourseId == model.CourseId, update);
        return true;
    }

    /// <summary>
    /// 取得一個課程
    /// </summary>
    public async Task<CourseModel> GetCourse(string courseId)
    {
        var result = await _collection.Find(x => x.CourseId == courseId).FirstOrDefaultAsync();
        return result;
    }

    /// <summary>
    /// 取得全部課程
    /// </summary>
    public async Task<List<CourseModel>> GetAllCourse()
    {
        var result = await _collection.Find(_ => true).ToListAsync();
        return result;
    }

    /// <summary>
    /// 刪除課程
    /// </summary>
    public async Task<bool> DeleteCourse(string courseId)
    {
        await _collection.DeleteOneAsync(x => x.CourseId == courseId);
        return true;
    }
}
```

接著就可以觀察結果，如下：

```c#
static async Task Main(string[] args)
{
    var setting = new CourseMongoDatabaseSetting
    {
        ConnectionString = "mongodb://localhost:27017",
        Database = "Sample",
        Collection = "Course"
    };

    // 建立 Repo 實體，並注入 setting
    var repository = new MongoRepository(setting);

    // 新增課程
    var createResult = await repository.CreateCourse(new CourseModel
    {
        CourseId = "123",
        CourseName = "英文",
        Teacher = "Amy",
        Students = new List<Student>
        {
            new Student { Id = 1, Name = "Bob" },
        }
    });

    // 更新課程教師(Teacher) & 課程名稱(CourseName)
    var updateResult = await repository.UpdateCourse(new UpdateCourseModel
    {
        CourseId = "123",
        CourseName = "國文",
        Teacher = "Andy"
    });

    // 取得課程
    var course = await repository.GetCourse("123");

    // 取得所有課程
    var courses = await repository.GetAllCourse();

    // 刪除課程
    var deleteResult = await repository.DeleteCourse("123");
}
```

原始碼：[連結](https://github.com/maxlin0523/UsingMongoClient)

### 參考

[使用 ASP.NET Core 與 MongoDB 建立 Web API](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-6.0&tabs=visual-studio)
