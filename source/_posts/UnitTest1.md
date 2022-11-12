---
title: UnitTest
catalog: true
date: 2021-09-11 22:02:36
subtitle:
header-img:
tags: 
- UnitTest
- General
---
# 單元測試
單元測試（Unit Test）就是將模組（dll）裡的最小單位分成若干個測試案例來進行測試，最小單位為一個方法，透過將方法的每個行為都拆分成一個個測試情境，並依情境進行單元測試撰寫確保所有行為的運行結果都符合開發者預期。

如：
```c#=
public class Test
    public string Greet(string name)
    {
        if (string.IsNullOrEmpty(name))
        {
            return "who are you? ";
        }
            
        return $"Hi {name}, nice to meet you.";
   }
}
```
我們可以將 `Greet` 方法可以分成三種情境，分別是：
1. 當參數 name 為 null 時，應回傳 who are you?
2. 當參數 name 為空字串時，應回傳 who are you?
3. 當參數 name 符合格式時，應回傳 Hi xxx, nice to meet you.

後續範例會以此方法示範

# 單元測試的特性

* 快速
> 一個單元測試為一個獨立情境不具備判斷邏輯且程式碼範圍小，所以必須要快

* 獨立
> 每個測試不可相依其他檔案系統或是類別也不可有邏輯判斷, 具備邏輯判斷意味著有多個情境會產生， 必須針對一個情境做測試，一個單元測試必須只關注一個行為。

* 可驗證性
> 測試後要能反應此次測試的結果

* 可重複性
> 每次重複測試的結果皆需相同

* 及時
> 寫完程式馬上就能進行測試

# 建立單元測試專案
首先，方法右鍵點擊 `建立單元測試`，或是由方案自行加入 `單元測試專案`　皆行

![](https://i.imgur.com/uRB1nPa.jpg)

預設框架為 MSTest 2，這邊按 `確定`

![](https://i.imgur.com/ereUtMP.png)

接下來就可以看到測試類別的內容，而測試主要都是用 Assert 這個類別提供的方法進行驗證 ([詳細用法](https://docs.microsoft.com/zh-tw/previous-versions/visualstudio/visual-studio-2010/ms245302(v=vs.100)?redirectedfrom=MSDN))，之後右鍵方法點選 `執行測試` 查看結果

![](https://i.imgur.com/nNNAfTt.jpg)

執行測試後會失敗，因為預設為 `Assert.Fail`，可以從測試總管 Test Explorer 檢視測試訊息

![](https://i.imgur.com/T8dzRr6.jpg)

# 如何撰寫單元測試？
撰寫方式遵循 `3A原則` 分別是 `Arrange`、`Act`、`Assert` 

定義如下：
* Arrange
> 測試場境。例如: 宣告測試目標類別 SystemUnderTest (SUT)、預期結果 Expected、測試替身...etc
* Act
> 呼叫測試目標方法並得出測試結果。
* Assert
> 驗證測試與預期結果。

# 實作

測試方法名稱以 `測試方法_行為的情境_行為反應的結果` 去命名。

寫法如下：
```c#=
[TestClass()]
public class TestTests
{
    [TestMethod()]
    public void Greet_當參數name為Null_應回傳Who_are_you()
    {
        // Arrange
        var sut = new Test();
        var expected = "who are you? ";

        // Act
        var actual = sut.Greet(null);

        // Assert
        Assert.AreEqual(expected, actual);
    }
}
```

接續把其他兩個情境補上

```c#=
[TestMethod()]
public void Greet_當參數name為Empty_應回傳Who_are_you()
{
    // Arrange
    var sut = new Test();
    var expected = "who are you? ";

    // Act
    var actual = sut.Greet(string.Empty);

    // Assert
    Assert.AreEqual(expected, actual);
}
```
以及
```c#=
[TestMethod()]
public void Greet_當參數name符合格式_應回傳正確訊息()
{
    // Arrange
    var sut = new Test();
    var expected = "Hi Max, nice to meet you.";

    // Act
    var actual = sut.Greet("Max");

    // Assert
    Assert.AreEqual(expected, actual);
}
```

這樣就完成 `Greet` 功能的測試了，之後點選上方 `測試` -> `執行所有測試` 就可查看結果

![](https://i.imgur.com/IkrNSFw.png)

# 參考
[[測試]單元測試的意義](https://dotblogs.com.tw/hatelove/2011/12/18/meaning-of-unit-test)
[[C#][Unit Test] 01. 軟體上線就等於今晚不用回家? 學"單元測試"可能有辦法挽救您的婚姻。](https://progressbar.tw/posts/11)