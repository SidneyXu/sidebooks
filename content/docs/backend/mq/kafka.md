---
weight: 1
title: Kafka
---

# Kafka

## 基本概念

- 基于拉模型，offset默认保存在Zookeeper上，也可以手动保存。
- 基于ZooKeeper实现动态扩容。
- 生产者不会立即发送消息，而是将消息积攒一批后发送。Broker将批量消息作为一个整体处理，在消费者消费时才将其解开。

## 使用方法

- 在设定consumer group的时候，只需要指明consumer数量即可，无需指定partition，consumer会自动进行rebalance。
- producer端发送消息时只需指定topic，无需指定partition，Kafka会把收到的message进行load balance，均匀的分布在这个topic下的不同的partition上。

## 集群架构

### 扩容

当添加新的partition的时，原partition里面的message不会重新进行分配，只有进入topic的新message才会通过load balance添加到新的partition。

### 重平衡

什么时候会发生重平衡

- Partition扩容。
- 消费者数量发生变更。
- 订阅的规则发生变更（例如基于正则订阅，新创建的主题也符合规则）。

重平衡时Kafka不可用，所以应通过提高`session.timout.ms`和`max.poll.interval.ms`的值和降低`heartbeat.interval.ms`的值来减少重平衡次数。