---
title: C# 陣列亂數排序(洗牌) - Fisher-Yates Shuffle
catalog: true
date: 2022-02-16 06:02:09
subtitle:
header-img:
tags: Algorithm
---

問題來源於之前在 interview 的 live demo 中，其中一個功能為 `洗牌` ，原本預期這功能是沒問題的，仔細做到才發現有點卡住，然而又是在 live 的情況，趕緊寫出一個很醜的寫法，雖然醜但至少先有個功能😂

### 假設情境為：將一個裝了 0 ~ 9 數字陣列做亂數排序(洗牌)

當時寫法如下：

```C#
Random random = new Random();

List<int> cards = new List<int>();

// 洗牌
while (true)
{
    if (cards.Count == 10)
    {
        break;
    }

    int value = random.Next(0, 10); // 亂數 0 ~ 9
    if (!cards.Contains(value))
    {
        cards.Add(value);
    }
}
```

這時間複雜度我也不知道怎麼算，反正是比 O(N) 差，且資料量越大越差，而後續查到有個洗牌演算法叫做 `Fisher-Yates Shuffle `，其時間複雜度為 O(N)。

### Fisher-Yates Shuffle

演算寫法如下：

```C#
Random random = new Random();

List<int> cards = new List<int> { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// 洗牌
for (int i = cards.Count - 1; i >= 0; i--)
{
    int transIndex = random.Next(0, i);  // 亂數 0 ~ (i - 1) 

    // 互換位置
    int temp = cards[transIndex];
    cards[transIndex] = cards[i];
    cards[i] = temp;
}
```

**Work Flow** 如下：

1. 從 cards[0] ~ cards[8] 中亂數取出一個元素，與 cards[9] 交換
2. 從 cards[0] ~ cards[7] 中亂數取出一個元素，與 cards[8] 交換
3. 從 cards[0] ~ cards[6] 中亂數取出一個元素，與 cards[7] 交換
...依此類推直到全部排完

### 其他洗牌寫法

也有看到搭配 LINQ 寫起來更簡潔的方法，如下：

```C#
List<int> cards = new List<int> { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// 洗牌
List<int> randomList = cards.OrderBy(x => Guid.NewGuid()).ToList();
```

### 參考
[Best way to randomize an array with .NET](https://stackoverflow.com/questions/108819/best-way-to-randomize-an-array-with-net)