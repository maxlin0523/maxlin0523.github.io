---
title: C# Boxing & Unboxing
catalog: true
date: 2021-08-03 21:09:17
subtitle:
header-img:
tags: C# 本事
---
# 前言
你是否曾疑惑： 
```C#=
var num = 3345678;
var result = $"我的電話: {num}";
```
為何這一段程式碼可以把 `int` 型別直接插進字串呢？

事實上

可以把 `$""` 看成 `String.Format` 的語法糖，在這種情況 `String.Format` 底層其實做了 Boxing 的動作，使 `int` 最終會轉為 `string` 後再填入字串裡，這樣做其實無形中產生了一些效能成本。

# 什麼是 Boxing & Unboxing

談 **Boxing & Unboxing** 前，要先了解 **實值型別** 與 **參考型別** 的存儲方式。
可以參考我的另一篇 -- [C# 實值型別 & 參考型別](/2021/08/03/value-reference/)


### Boxing & Unboxing：
`Boxing` 是指 `實值型別` 轉為 `參考型別` 中間的轉換過程，反之為 `Unboxing`，也就是 `參考型別` 轉為 `實值型別` 的轉換過程。

※ 若兩者都為 `實值型別` 或 `參考型別`，就不會發生此情況。

說明如下：
```C#
int i = 123;
object o = i; // boxing ( 實質 => 參考 )
int j = (int)o // unboxing ( 參考 => 實質 )
```
在轉換過程中，實值型別的值原本存於 `Stack` 記憶體裡，但經轉換成參考型別 `object` ，會將值轉存於 `heap` 記憶體，反之亦然，這樣一來一往無形中就會產生效能損耗。

### String.Format
再回頭看 `String.Format` 這個函式，其中一個多載為：
`public static string Format (string format, params object?[] args)`
可以看到傳入參數是 `object` 型別，也就代表不管傳入參數的資料型態為何，最終都將被強轉成 `object`， 所以，當傳入 `實值型別` 參數時，就會發生 `Boxing`。

測試如下：
```C#
void Main()
{
	// arrange
	var sw = new Stopwatch();
	var count = 10000000;	
	
	// 裝箱測試
	sw.Start();	
	for (int i = 0; i < count; i++)
	{
		var result = String.Format("{0}{1}{2}", i, i, i);		
	}	
	sw.Stop();
	
	Console.WriteLine($"Boxing Test: {sw.ElapsedMilliseconds} ms");
		
	// 不裝箱測試
	sw.Restart();
	for (int i = 0; i < count; i++)
	{
		var result = String.Format("{0}{1}{2}", i.ToString(), i.ToString(), i.ToString());
	}
	sw.Stop();
	
	Console.WriteLine($"No Boxing Test: {sw.ElapsedMilliseconds} ms");
    
    // Output
    // Boxing Test: 4919 ms
    // No Boxing Test: 4673 ms
}
```

# 參考
[[ASP<span>.</span>NET] Boxing(裝箱) 與 UnBoxing(拆箱) 測試](https://dotblogs.com.tw/joysdw12/2013/08/06/asp-net-boxing-unboxing)
[那些年String.Format中的Boxing和UnBoxing](https://isdaniel.github.io/stringformat-compare/)
