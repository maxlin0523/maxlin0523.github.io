---
title: Apache Kafka 
catalog: true
date: 2022-06-01 18:16:18
subtitle:
header-img:
tags:
- Message Queue
---
## 簡介
Kafka 最初是 Linkedin 開發，後 Confluent 維護與開發，是一套消息代理(Message broker) 的系統，並具有高吞吐、可擴充、高可用、持久的特性。

## 何謂消息代理？
擔任各服務之間的中介層，應用在溝通、信息的傳遞，並代理服務傳送的訊息，流程如下：

![](https://i.imgur.com/DgPejAs.png)

Producer 生產一個訊息，會存在一個訊息佇列裡(在此指 kafka)，Consumer 會再透過訊息佇列去消費訊息。

常用來解決複雜的服務關係，有效達到解耦，kafka 也有以下好處：
1. 可用性：kafka 可以是一個分散式架構，如：Replication
2. 可擴展性: 架構進行擴展縮放無需停機，如：Cluster
3. 耐用性： 所有訊息都是存放於硬碟裡
4. 性能：由於在磁碟讀寫都是循序的，速度可比 `RAM` 時間快，並有著高吞吐量可達到巨量緩衝作用

## 利用 Docker 建置
#### Docker-Compose
GUI 使用者介面：[kafka-tool](https://www.kafkatool.com/)
```yaml=
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_ADVERTISED_HOST_NAME: localhost
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
```

### C#
下載 Nuget `Confluent.Kafka`

#### Producer
```c#
// kafka 位置
var config = new ProducerConfig { BootstrapServers = "localhost:9092" };

// Producer
using (var p = new ProducerBuilder<Null, string>(config).Build())
{
    try
    {
        await p.ProduceAsync("max-topic", new Message<Null, string> { Value = "a" });
        await p.ProduceAsync("max-topic", new Message<Null, string> { Value = "b" });
        await p.ProduceAsync("max-topic", new Message<Null, string> { Value = "c" });
        await p.ProduceAsync("max-topic", new Message<Null, string> { Value = "d" });
        await p.ProduceAsync("max-topic", new Message<Null, string> { Value = "e" });
    }
    catch (ProduceException<Null, string> e)
    {
        Console.WriteLine($"Delivery failed: {e.Error.Reason}");
    }
}
```

#### Consumer
```c#=
var conf = new ConsumerConfig
{
    GroupId = "test-consumer-group",
    BootstrapServers = "localhost:9092",
    AutoOffsetReset = AutoOffsetReset.Earliest
};

using (var c = new ConsumerBuilder<Ignore, string>(conf).Build())
{
    c.Subscribe("max-topic"); // 訂閱 Topic

    CancellationTokenSource cts = new CancellationTokenSource();

    try
    {
        while (true)
        {
            try
            {
                var cr = c.Consume(cts.Token);
                Console.WriteLine($"Consumed message '{cr.Value}' at: '{cr.TopicPartitionOffset}'.");
            }
            catch (ConsumeException e)
            {
                Console.WriteLine($"Error occured: {e.Error.Reason}");
            }

            Thread.Sleep(1000); // 一秒消費一次
        }
    }
    catch (OperationCanceledException)
    {
        c.Close();
    }
}
```
























## 參考：
[Kafka 維基百科](https://zh.wikipedia.org/zh-tw/Kafka)

[Apache Kafka 概述](https://morosedog.gitlab.io/kafka-20201117-kafka-1/)

[confluentinc/confluent-kafka-dotnet](https://github.com/confluentinc/confluent-kafka-dotnet)

[Apache Kafka 介紹](https://medium.com/@chihsuan/introduction-to-apache-kafka-1cae693aa85e)