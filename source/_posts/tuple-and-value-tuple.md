---
title: C# Tuple & ValueTuple
catalog: true
date: 2024-01-05 00:13:14
subtitle:
header-img:
tags: 
- C#
---
C# 中 Tuple & ValueTuple 兩者使用方式相似，都是將多個值組合在一起的結構，避免因為需要多個變數而建立額外的類別。

## 差異

|          | ValueTuple               |          Tuple           |
| -------- | ------------------------ |:------------------------:|
| 類型     | 實值(Value)類型          |   參考(Reference)類型    |
| 效能     | 宣告範圍結束釋放，較輕量 | 需透過 GC 等待釋放 |
| 元素     | 8個以上                  |         限制 8個         |
| 元素名稱 | 可自訂名稱               |  只能呼叫 `Item<Index>`  |

## 實作

使用方式

```csharp
static void Main(string[] args)
{
    // 創建方式
    var valueTuple = (good: 1, Bad: "2",Sad : 3.3333); // ValueTuple
    var tuple = Tuple.Create(1, "2", 3.3333);          // Tuple
    
    // 呼叫元素方式
    var valueTupleItem = valueTuple.good; // ValueTuple
    var tupleItem = tuple.Item1;          // Tuple


    Console.WriteLine($"ValueTuple：{valueTuple.GetType()}");
    Console.WriteLine($"Tuple：{tuple.GetType()}");
    // Output：
    // ValueTuple：System.ValueTuple`3[System.Int32,System.String,System.Double]
    // ValueTuple：System.Tuple`3[System.Int32,System.String,System.Double]
}
```

如果你只是需要一個輕量的結構來組合多個值，不需進行複雜的操作，那 ValueTuple 可能更適合