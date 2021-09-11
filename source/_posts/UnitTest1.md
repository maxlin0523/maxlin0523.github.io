---
title: 單元測試 Part 1
catalog: true
date: 2021-09-11 22:02:36
subtitle:
header-img:
tags: UnitTest
---
### 什麼是單元測試？
單元測試（Unit Test）就是將模組（dll）裡的最小單位分成若干個測試案例來進行測試，最小單位為一個 cs 檔也就是一個類別，透過將類別裡的每個邏輯都拆分成獨立的一個測試情境，並進行測試確保所有邏輯的運行結果都符合開發者預期，例如：
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
我們可以將 `Greet` 方法可以分成三種情境，分別是
1. 當參數 name 為 Null 時，應回傳 who are you?
2. 當參數 name 為空值字串時，應回傳 who are you?
3. 當參數 name 符合格式時，應回傳 Hi {name}, nice to meet you.

知道了每個測試情境後就可開始建立單元測試專案

### 建立單元測試專案
首先，方法右鍵點擊`建立單元測試`，或是由方案自行加入都行
![](https://i.imgur.com/uRB1nPa.jpg)

預設框架為 MSTest 2，這邊按`確定`
![](https://i.imgur.com/ereUtMP.png)

接下來就可以看到測試類別的內容，而測試主要都是用 Assert 這個類庫提供的方法進行驗證，請先閱讀（[連結](https://docs.microsoft.com/zh-tw/previous-versions/visualstudio/visual-studio-2010/ms245302(v=vs.100)?redirectedfrom=MSDN)），之後右鍵方法點選 `執行測試` 查看結果
![](https://i.imgur.com/nNNAfTt.jpg)

執行測試後會失敗，因為預設為 Assert.Fail，可以從測試總管 Test Explorer 檢視測試訊息
![](https://i.imgur.com/T8dzRr6.jpg)

細節很多沒法全部描述，大家在自行摸索一下

### 如何撰寫單元測試？
撰寫方式遵循 `3A原則` 分別是 Arrange、Act、Assert 而定義如下：
* Arrange：測試場境。例如: 宣告測試目標物件 SystemUnderTest(簡稱SUT)、預期結果 Expected、測試替身...etc
* Act：呼叫被測試成員並得出測試結果。
* Assert：驗證測試與預期結果。

具體寫法如下：
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
public void Greet_當參數name符合格式_應回傳nice_to_meet_you訊息()
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

下篇將提到 三種對測試目標的測試方式、單元測試特性以及測試帶給產品的好處
