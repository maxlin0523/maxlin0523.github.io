---
title: 加密 & 雜湊
catalog: true
date: 2022-11-15 23:12:44
subtitle:
header-img:
tags: 
- 其他
---

## 雜湊(Hash)
雜湊又稱哈希，雜湊值為一段明文(`plain text`)透過雜湊函數(`hash function`)所演算出來的一個值，並且過程是 `單向` 也就是 `不可逆` 的，常用於密碼存儲的部分。

例如：
用戶密碼就以雜湊值的形式儲存於資料庫，當用戶登入時，將輸入明文密碼以同樣的雜湊函數演算完去跟資料庫的密碼雜湊值比對，登入成功代表密碼所算出的雜湊值與資料庫儲存的雜湊值相同

如此一來，當資料庫受到攻擊時，密碼也就以雜湊值的形式外流，而雜湊又是 `不可逆` 的，就使得真正的資料無法被竊取，當然，也可在中途將明文密碼多添加一些額外規則來增加安全性。

範例：

`哈哈` 經 `sha-1` 雜湊函數所演算的雜湊值

![](https://i.imgur.com/3fAMKAC.jpg)

[**連結**](http://www.sha1-online.com/)

### 雜湊的特性
* 一個明文通常只有一個雜湊值
* 雜湊值通常無法逆推明文
* 雜湊函數必須演算很快
* 兩段明文就算很相似，其雜湊值通常都大不相同

### 常見的雜湊函數
* MD5
* SHA-1
* Tiger

### 雜湊碰撞(Hash Collision)
代表該雜湊函數出現兩段明文演算出 `相同雜湊值`　的情況，當出現這種情況，如：`A` 跟 `B` 擁有相同雜湊值，可能的風險為：你使用了 `A` 去查詢，結果回傳了 `B` 的資料。

## 加密(Encryption)
加密主要的用途為 `保護敏感數據`，如密碼、帳號、信用卡、帳單...等數據，當把這些明文透過加密演算法算出密文，而密文通常都可以解密回原資料，主要是為了防範資料在傳輸過程受到攻擊攔截，當攻擊者攔截到資料，卻已是加密後的密文而無法達到目的，常見加密方式有以下兩種：

### 對稱加密(Symmetric Encryption)
透過一組對稱加密演算法搭配一組密鑰來完成加解密功能，簡單來說，假設我用 `AES` 對稱加密演算法搭配一組叫 `Max` 的密鑰來加密，那如果要解密的話就需用 `AES` 對稱加密演算法搭配同樣叫 `Max` 的密鑰來解密。

![](https://i.imgur.com/MmTK1GV.png)


### 非對稱加密(Asymmetric Encryption)

透過公鑰與私鑰來進行非對稱加密前，先記住幾點：
* 公鑰是廣泛公開且大家都知道
* 私鑰只有當事人知道
* 公私鑰是一對的，當經過公鑰加密後需拿配對的私鑰才能解密
* 每個人都有屬於自己的公鑰與私鑰

假設我要發信件給 Max，我會利用 Max 公開出來的 `公鑰` 加密傳給他，後續他才可用自己的 `私鑰` 解密查看信件，但問題是若有攻擊者拿 Max 的 `公鑰` 發病毒該怎麼辦？要怎麼確認是我這個`已信任`的人發信的？這時候會使用到 **數位簽章(Digital Signature)** ，就需要再做一次相似的步驟，在發信時再多傳一份由我的 `私鑰` 與 `信件資料` 所產生出的加密簽章過去，之後，Max 再拿我公開出來的 `公鑰` 進行驗證簽章即可確認完成。

![](https://i.imgur.com/XgLE1hv.png)

### 參考
[Hashing vs Encryption – What Are the Difference?](https://www.clickssl.net/blog/difference-between-hashing-vs-encryption)

[一次搞懂密碼學中的三兄弟 — Encode、Encrypt 跟 Hash](https://medium.com/starbugs/what-are-encoding-encrypt-and-hashing-4b03d40e7b0c)

[Hash Table：Intro(簡介)](http://alrightchiu.github.io/SecondRound/hash-tableintrojian-jie.html)

[基礎密碼學(對稱式與非對稱式加密技術)](https://medium.com/@RiverChan/%E5%9F%BA%E7%A4%8E%E5%AF%86%E7%A2%BC%E5%AD%B8-%E5%B0%8D%E7%A8%B1%E5%BC%8F%E8%88%87%E9%9D%9E%E5%B0%8D%E7%A8%B1%E5%BC%8F%E5%8A%A0%E5%AF%86%E6%8A%80%E8%A1%93-de25fd5fa537)