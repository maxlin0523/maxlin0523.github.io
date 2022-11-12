---
title: C# Value and Reference Type
catalog: true
date: 2021-08-03 21:03:01
subtitle:
header-img:
tags: C#
---
# 前言
在 C# 世界裡，資料型態分為 **實值型別（Value Type）** 與 **參考型別（Reference Type）**，例如像 `int`、`char`、`byte` ...等屬於實值型別，`class`、`object`、`string` 、`List` ...等屬於參考型別，辨別方法為參考類型需使用 `new` 關鍵字實體化，但請記得 `string` 也屬於參考型別（詳細請[參閱](https://medium.com/ninas-note/c-%E5%AF%A6%E8%B3%AA%E5%9E%8B%E5%88%A5-value-type-vs%E5%8F%83%E8%80%83%E5%9E%8B%E5%88%A5-reference-type-64ba5cf8bf8b)），而兩者主要的差異在於存儲方式。

#### 實值型別：
變數和值都會存放於 `堆疊（Stack）` 記憶體，當變數指派一個實值型別的值例如 `var a = 123`，則會在 `Stack` 新增一塊區域儲存，若 `var b = a`，則會在複製一份 `a` 的值指派給 `b`，彼此間互相獨立。

如下：
![](https://i.imgur.com/H1HJAsu.png)



#### 參考型別：
變數會存放於 `堆疊（Stack）` 記憶體，而值會存放於 `堆積（Heap）` 記憶體中的某個位址，當變數指派一個參考型別的值，變數則會指向該 `Heap` 記憶體位址，例如 `var a = new List<int>()`　當中 `a` 會指向該 `List` 的記憶體位址進而得到物件，而當 `var b = a`，則會在 `Stack` 新增一塊空間給 `b` 而值則指向和 `a` 相同的記憶體位址，彼此共用物件，當任一方改動，則會影響到另一方。

如下：
![](https://i.imgur.com/7xlC01B.png)


# 驗證

######  實值型別：
```C#=
void Main()
{
	// 實值型別
	var a = 123;
	var b = a;

	b -= 100;

	Console.WriteLine("a = " + a);
	Console.WriteLine("b = " + b);
    
	// Output
	// a = 123
	// b = 23
}
```
`a`、`b` 彼此間互相獨立，互不影響。

######  參考型別：
```C#=
void Main()
{
	// 參考型別
	var a = new List<int>() { 1,2,3 };
	var b = a;

	b.Add(4);
	b.Add(5);
	b.Add(6);
	a.Remove(6);

	Console.WriteLine("a = " + string.Join(",", a));
	Console.WriteLine("b = " + string.Join(",", b));
    
	// Output
	// a = 1,2,3,4,5
	// b = 1,2,3,4,5
}
```
`a`、`b` 彼此共用物件，當任一方改動，都會影響到另一方。
