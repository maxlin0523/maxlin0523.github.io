---
title: 基礎 LuaScript 語法
catalog: true
date: 2021-10-16 01:48:14
subtitle:
header-img:
tags: Lua
---
最近接觸了 Redis 腳本有使用到 Lua，簡單紀錄下基礎語法。

#### 常用資料型態
|  資料型態  |                 備註                  |
|:----------:|:-------------------------------------:|
|   `nil`    |                 null                  |
| `boolean`  | false 和 nil 為假，true 和非 nil 為真 |
|  `number`  |           包含整數與小數位            |
|  `string`  |     以雙引號 "" 或單引號 '' 包覆      |
|  `table`   |   類似 dictionary 或當作 array 使用   |
| `function` |                 函式                  |


#### 運算符號

| 運算符 |    備註    |
|:------:|:----------:|
|  `+`   |    加法    |
|  `-`   | 減法或負號 |
|  `*`   |    乘法    |
|  `/`   |    除法    |
|  `％`  |    餘數    |
|  `＾`  |    次方    |

#### 操作符號
|  運算符  |      備註      |
|:--------:|:--------------:|
|   `==`   |      等於      |
|   `~=`   |     不等於     |
|   `>`    |      大於      |
|   `<`    |      小於      |
|   `>=`   |    大於等於    |
|   `<=`   |    小於等於    |

#### 變數
可一次宣告多個變數，沒指派值的會是 `nil`

```lua 
local a = 1
local b, c, d = 2, 3

print(a)
print(b)
print(c)
print(d)
-- Output:
-- 1
-- 2
-- 3
-- nil
```


---

#### **if else**

```lua
if(boolean) then
   -- do...
else if(boolean) then
   -- do...
else 
   -- do...
end
```


---

#### **Loop 系列**

* **for loop**
i 從 `0` 開始迴圈每次遞增 `1` 直到 `i <= 3`

```lua
for i = 0, 3, 1 do
  print(i)
end
-- Output:
-- 0
-- 1
-- 2
-- 3
```

* **repeat until**
從 `repeat` 內開始跑迴圈直到 `until` 條件式成立就跳出

```lua
local i = 1
local count = 3
repeat
  print(i)
  i = i + 1
until i == count
-- Output:
-- 1
-- 2
```

* **while do**
從 `while` 內開始跑迴圈直到條件式不成立就跳出

```lua
local i = 1
local count = 3
while i <= count do
  print(i)
  i = i + 1
end
-- Output:
-- 1
-- 2
-- 3
```

* **ipairs**
`ipairs` 會按照陣列索引順序開始跑，直到 `value` 出現 `nil` 的情況

```lua
local a = { [1] =  "one", [2] = "two", [4] = "four" }
for index, value in ipairs(a) do
    print(index .. ":" .. value)
end
-- Output:
-- 1:one
-- 2:two
```

* **pairs**
`pairs` 會按順序跑完全部陣列

```lua
local a = {
  [1] =  "one",
  [2] = "two",
  [4] = "four" ,
  dog = "狗",
  ["貓"] = "cat"
  }
for index, value in pairs(a) do
    print(index .. ":" .. value)
end
-- Output:
-- 1:one
-- 2:two
-- 4:four
-- dog:狗
-- 貓:cat
```


---

#### Function 系列

* **具有回傳值**
使用 `return` 回傳

```lua
function greet(name)
   local result = ""
   if (name == 'Max') 
   then
      result = 'Hello ' .. name
   else
      result = 'Who are you'
   end

   return result;
end

print(greet('Max'))
-- Output:
-- Hello Max
```

* **具有多回傳值**
使用 `return` 回傳，且用`逗號`分隔回傳值
```lua
function calculate(a, b)
  return a + b, a - b
end

local a, b = calculate(5, 23)
print('a + b = ' .. a)
print('a - b = ' .. b)
-- Output:
-- a + b = 28
-- a - b = -18
```

* **不具回傳值**
相當於 `void`

```lua
function greet(name)
   if (name == 'Max') 
   then
      print('Hello ' .. name)
   else
      print('Who are you')
   end
end

greet('someone')
-- Output:
-- Who are you
```

* **傳入任意參數**
傳入參數以`逗號`分隔

```lua
function add(...)
  local total = 0
  for index, value in ipairs{...} do
    total = total + value
  end
  
  return total
end

print(add(1,2,3))
-- Output:
-- 6
```

---

#### 小試身手

題目：[LeetCode 1. Two Sum](https://leetcode.com/problems/two-sum/)

```lua=
function TwoSum(nums, target)
  local table = { }
  for index, value in ipairs(nums) do
    if(table[target - value])
    then
      return table[target - value], index
    end
    table[value] = index
  end
end

local nums = {3,2,4} -- Sample
local target = 7 -- Sample
local one,two = TwoSum(nums, target)

print('[' .. one .. ',' .. two .. ']')

-- Input: nums = [3,2,4], target = 7

-- Output: [1,3]    
-- ※ Lua 的陣列索引從 1 起跳，所以原本答案是 [0,2] 會變為 [1,3]
```

# 參考
[Lua 教程](https://www.runoob.com/lua/lua-tutorial.html)
[撰寫和使用函式 (Function)](https://opensourcedoc.com/lua-programming/function/)
[Lua | pairs與ipairs的差異](https://emilykevin.pixnet.net/blog/post/249421762-lua-%7C-pairs%E8%88%87ipairs%E7%9A%84%E5%B7%AE%E7%95%B0)
[Tag: Lua | Level Up](https://larrynung.github.io/tags/Lua/)