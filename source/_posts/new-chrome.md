---
title: Google Chrome 建立獨立環境
catalog: true
date: 2021-11-18 01:11:47
subtitle:
header-img:
tags: Chrome
---

最近需回家處理工作的事，而我通常都是先登出自己的筆記軟體帳號，然後再登入公司筆記軟體帳號來查看我的工作項目，但這樣一來每次都要切來切去有點麻煩

這時如果有兩個獨立的 Chrome，一個自己平常使用，另一個則是工作使用該有多好

## 做法

### 第一步：建立資料夾

獨立一個資料夾給新的 Google Chrome 使用

如：

建立資料夾，路徑：`D:\work` 

![](https://i.imgur.com/AhczMWu.jpg)

### 第二步：建立一個 Work 的 Google Chrome 

將桌面 Chrome 執行檔複製一份並取個喜歡的名字，這邊我取 `Work`

如：

![](https://i.imgur.com/0n7CLGs.png)

### 第三步：右鍵 Work 點選內容後更改目標路徑

右鍵 Work 點選內容

如：

![](https://i.imgur.com/wf8QTAW.jpg)

之後，於`目標`路徑結尾 `空一格` 再新增路徑 `--user-data-dir="<path>"`，我這邊是 `--user-data-dir="D:\work"`

如：

![](https://i.imgur.com/ExmMdZj.jpg)

### 第四步：開啟 Work 瀏覽器查看

應該就會是全新獨立的瀏覽器了

如：

![](https://i.imgur.com/TyAEWXQ.png)

# 參考

[Chrome 技术篇-一台电脑设置多个独立chrome方法实例演示，chrome独立多开技术，chrome多开并加载原chrome的数据方法](https://blog.csdn.net/qq_38161040/article/details/88824517)
