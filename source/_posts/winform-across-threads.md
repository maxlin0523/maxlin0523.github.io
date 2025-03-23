---
title: Winform 控制項跨執行緒作業無效
catalog: true
date: 2022-04-09 23:37:03
subtitle:
header-img:
tags: Debug
---
在開發 Winform 時，碰到這種例外狀況：`跨執行緒作業無效`，原因是：當對 UI 控制項 `更動的執行緒`與 `UI 控制項的執行緒`不相同，這會導致對 UI 控制項進行不安全的調用，從而引發此例外錯誤。

### 示例

簡單寫了個介面，功能為：點擊`計數`按鈕時，灰色區域要顯示 0 ~ 9 的數字(如右)

![](https://i.imgur.com/n6Redh8.png) ![](https://i.imgur.com/5TNkASm.png)

計數按鈕程式碼：

```c#
/// <summary>
/// 計數
/// </summary>
private void BtnCount_Click(object sender, EventArgs e)
{
    for (int i = 0; i < 10; i++)
    {
        LblView.Text += $"{i} \r\n";
    }
}
```

若改為由不同的執行緒處理時：

```c#
/// <summary>
/// 計數
/// </summary>
private void BtnCount_Click(object sender, EventArgs e)
{
    Thread thread = new Thread(() =>
    {
        for (int i = 0; i < 10; i++)
        {
            LblView.Text += $"{i} \r\n";
        }
    });

    thread.Start();
}
```

點擊時，則會引發錯誤

![](https://i.imgur.com/0Lpzqqw.png)

### 解決方式

首先建立宣告委派 `DelSetLalView` 及要委託執行的方法(這邊指設定 `LblView.Text` 的這個行為，後續會在宣告 `SetLblView` 方法)，如下：

```C#
/// <summary>
/// 委派 SetLalView
/// </summary>
public delegate void DelSetLblView(int num);

// ...

private void SetLblView(int num)
{
    LblView.Text += $"{num} \r\n";
}
```

之後宣告 `SetLblViewSafely` 方法，並且這方法會判斷是否需進行安全調用，如下：

```C#
private void SetLblViewSafely(int num)
{
    // 若為不同執行緒，則 InvokeRequired 會為 ture，表示需進行安全的調用
    if (this.InvokeRequired)
    {
        DelSetLblView del = new DelSetLblView(SetLblView);
        this.Invoke(del, new object[] { num });
    }
    else
    {
        SetLblView(num);
    }     
}
```

完整程式碼：

```C#
/// <summary>
/// 委派 SetLalView
/// </summary>
public delegate void DelSetLblView(int num);

public Form1()
{
    InitializeComponent();  
}

/// <summary>
/// 計數
/// </summary>
private void BtnCount_Click(object sender, EventArgs e)
{
    Thread thread = new Thread(() =>
    {
        for (int i = 0; i < 10; i++)
        {
            SetLblViewSafely(i);
        }
    });

    thread.Start();
}

private void SetLblViewSafely(int num)
{
    // 若為不同執行緒，則 InvokeRequired 為 ture，表示需進行安全的調用
    if (this.InvokeRequired)
    {
        DelSetLblView del = new DelSetLblView(SetLblView);
        this.Invoke(del, new object[] { num });
    }
    else
    {
        SetLblView(num);
    }     
}

private void SetLblView(int num)
{
    LblView.Text += $"{num} \r\n";
}
```

如此一來，即可在不同執行緒的情況下，安全改動 UI 控制項囉！

若懶得寫得像上述那麼繁瑣，改成這樣就好：

```C#
private void SetLblViewSafely(int num)
{
    Action action = () => LblView.Text += $"{i} \r\n";

    // 若為不同執行緒，則 InvokeRequired 為 ture，表示需進行安全的調用
    if (this.InvokeRequired)
    {
         this.Invoke(action);
    }
    else
    {
        action.Invoke();
    }     
}
```

### 參考

[C# Development | 煩人的跨執行緒作業無效 : 如何跨執行緒控制 UI 元件](https://medium.com/delightlearner/c-development-%E7%85%A9%E4%BA%BA%E7%9A%84%E8%B7%A8%E5%9F%B7%E8%A1%8C%E7%B7%92%E4%BD%9C%E6%A5%AD%E7%84%A1%E6%95%88-%E5%A6%82%E4%BD%95%E8%B7%A8%E5%9F%B7%E8%A1%8C%E7%B7%92%E6%8E%A7%E5%88%B6-ui-%E5%85%83%E4%BB%B6-41c7b129f47a)

[如何 Windows Forms .net) 對控制項進行安全線程呼叫](https://docs.microsoft.com/zh-tw/dotnet/desktop/winforms/controls/how-to-make-thread-safe-calls?view=netdesktop-6.0)