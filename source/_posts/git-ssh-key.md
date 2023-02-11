---
title: Github 使用 SSH-Key
catalog: true
date: 2023-01-12 22:01:48
subtitle:
header-img:
tags:
---
是一種 Github 的認證機制，用途是允許使用者在不提供密碼的情況下使用 SSH 進行遠端存取。

主要使用 `非對稱` 的方式來驗證是否為合法的使用者，所以會需要產生出一對金鑰，分別為：
* 公鑰(`Public Key`)：存放於 Github Server
* 私鑰(`Private Key`)：存放於使用者的本機

當本機對遠端進行操作時，就會將私鑰與公鑰進行`匹配`來驗證是否合法。

### 產生 SSH-Key

打開 Terminal 輸入：

`$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`

之後出現一些 option 就繼續按 `Enter`

如下：
![](https://i.imgur.com/OJYE0NQ.png)

出現以上畫面就代表產生成功。

## 取得 Public Key

打開 Terminal 輸入：

`$ cd ~/.ssh`

可以查看到 public key

![](https://i.imgur.com/fxClwd4.png)

查看公鑰(末尾應該會是輸入的 `Email`)

![](https://i.imgur.com/L8ezJ97.png)

## 將 Public Key 放到 Github Server

進到 Github 點出以下頁面：

![](https://i.imgur.com/1ZbvNv3.png)

進入該頁面點擊 `New SSH Key`

填好 `Title` 後，並將公鑰貼在 `Key` 欄位

![](https://i.imgur.com/aXRT9x7.png)


之後點擊 `Add SSH Key` 輸入完密碼即可啟用。


### 參考

[SSH 金鑰：免密碼登入遠端主機、傳遞檔案](https://noob.tw/ssh-key/)

[4.3 伺服器上的 Git - 產生你的 SSH 公鑰](https://git-scm.com/book/zh-tw/v2/%E4%BC%BA%E6%9C%8D%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%A2%E7%94%9F%E4%BD%A0%E7%9A%84-SSH-%E5%85%AC%E9%91%B0)

[Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

