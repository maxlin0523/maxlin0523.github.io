---
title: 'Hexo counts visitors'
catalog: true
date: 2021-07-30 20:47:12
subtitle:
header-img: "img/Architecture.jpeg"
tags: Hexo
---
本篇採用 `不蒜子計數器` 。

### 第一步：引用腳本
將網站標籤 `<Head>` 引入：
```javascript=
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

### 第二步：網站觀看數

將想要顯示網站計數的片段（footer.ejs）貼上：
```javascript=
總訪問量: <span id="busuanzi_value_site_pv"></span>次
```
或
```javascript
總訪客數: <span id="busuanzi_value_site_uv"></span>次
```

算法
pv： 單用戶點擊 n 篇文章，觀看數增加 n 次
uv： 單用戶點擊 n 篇文章，觀看數增加 1 次

### 第三步：文章觀看數

將想要顯示文章計數的片段（header.ejs）貼上：
```javascript=
文章觀看數: <span id="busuanzi_value_page_pv"></span>
```

done.


當然，後續也可添加一些 CSS 裝飾

### 範本
我的網頁
###### tags: 文章觀看數
![](https://i.imgur.com/17kN728.jpg)
###### tags: 網站觀看數（pv）
![](https://i.imgur.com/dkOnxR8.jpg)


### 原理
將域名生成唯一碼後傳入第三方服務計數，每次點擊就計數一次，只要域名不變，訪問數就會一直累計。


※ 要注意的是在本地端開啟時，網站訪問量數字會很大，因為大家本地端都是由 `http://localhost:4000/` 開啟，所以生成的唯一碼都相同，此情況在部屬後會恢復正常。


### 參考
[不蒜子计数器的使用](https://blog.csdn.net/weixin_46247581/article/details/105848853)