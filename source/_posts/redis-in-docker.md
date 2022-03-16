---
title: Docker 佈署 Redis
catalog: true
date: 2021-10-09 14:42:46
subtitle:
header-img:
tags:
- Redis
- Docker
---
## 流程說明

首先準備好相關 `Docker` 工具，如：Docker CLI，若不習慣指令操作的朋友(就是我XD)，可下載 Docker Desktop，比較方便觀察。

#### 第一步：獲取 image 映像檔

執行 `dokcer pull redis` 命令從遠端獲取 image 映像檔

載好後，可執行指令 `docker images` 查看所有 images，會看到剛剛 pull 的映像檔

如：

![](https://i.imgur.com/RztlmAl.jpg)

#### 第二步：將 redis image 部署於 container 
執行 `docker run --name <container_name> -p <external_port>:6379 -d redis:latest` 命令

`<>` 中的值由自己去定義。如：執行指令 `docker run --name myredis -p 4442:6379 -d redis:latest` 建立成功後，便可在本機透過 `127.0.0.1:4442` 進行串接。

執行指令 `docker ps` 查看正在運行的容器，就會看到建立的 Redis server

如：

![](https://i.imgur.com/04O3Mv7.jpg)

#### 第三步：進入容器 redis 終端機

執行　`docker exec -it <container_name> redis-cli`

進入容器中的 redis 終端後，即可下 Redis CLI 操作

如：

建立 key-value 為 max 3345678 的資料，之後在 get 該 key 便得出此 value。

![](https://i.imgur.com/oKJnpP2.png)

# Redis CLI 學習資源

[基本 Redis 操作命令(中文版 Redis 2.8)](http://doc.redisfans.com/)

[官網操作命令大全(最新)](https://redis.io/commands)

# Redis GUI

GUI 介面可讓使用者更直覺的進行資料操作，我選用以下 GUI：

[Another-Redis-Desktop-Manager](https://github.com/qishibo/AnotherRedisDesktopManager/)

畫面如：

![](https://i.imgur.com/IPpCRjQ.png)
