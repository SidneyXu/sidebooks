---
weight: 100
title: 消息中间件
bookCollapseSection: true
---

# 消息中间件

## 各种消息队列中间件的对比

性能
* Kafka：几十万级别。通过批量和异步获得高吞吐量，时延也较高。如果每秒数据量没那么多情况下时延会很高，不太适合在线业务场景。在大数据和流计算方面生态优秀。允许n-1个节点失败。
* RocketMQ：几十万级别。时延低。
* RabbitMQ：十万级别。消息堆积会引起性能下降。

基本组成
* Kafka：Producer、Consumer、Consumer Group、Broker、Topic、Partition
  - **Topic**：由多个Partition组成。
  - **Partition**：保存在Broker上，是Topic的物理形态。在不同Broker上可以建立Partition副本来实现高可用，这样原始Partition为Leader，其它为Follower。
  - **Consumer Group**：由多个Consumer组成。
  - **ZooKeeper**：用于实现动态扩容。
* RocketMQ：Producer、Consumer、Consumer Group、Topic、Queue
  - **Broker**：Broker间可以搭建主从架构实现高可用。
  - **Topic**：由多个Queue组成。
  - **Queue**：保存在Broker上，是Topic的物理形态。
  - **Consumer Group**：由多个Consumer组成。
  - **nameserver**：用于实现动态扩容。
* RabbitMQ：Producer、Consumer、Exchange、Channel、Queue
  - **Channel**：实现TCP重用，Producer和Consumer通过Channel连接RabbitMQ。
  - **Exchange**：交换机决定了消息该如何发送给Queue。

消费策略
* Kafka
  - **消息顺序**：Partition上有序。
  - **推拉模型**：基于长轮询的拉模型。
  - **消息删除**：无论消息是否被消费，只有到期后消息才会被删除。
  - **消费次数**：支持最多一次、最少一次、正好一次。正好一次是通过transaction幂等使消息在broker端只落地一次，在消费端则不保证。
  - **消费模型**：一条消息只能被同个Consumer Group中的一个Consumer消费。
  - 分配多个Consumer Group可以实现一条消息被多次消费。
  - **消息回溯**：可按Offset进行回溯。
* RocketMQ
  - **消息顺序**：Queue上有序。
  - **延时消息**：基于一定时间间隔。
  - **推拉模型**：都支持，但推模型实际也是通过拉模型来实现的。
  - 可以通过Tag在Broker层实现消息过滤。
  - 支持事务消息。
  - **消费模型**：一条消息只能被同个Consumer Group中的一个Consumer消费。 分配多个Consumer Group可以实现一条消息被多次消费。
  - **消息回溯**：可按秒进行回溯。
* RabbitMQ
  - **延时消息**：通过死信队列实现任意间隔。
  - **推拉模型**：都支持。
  - **消费次数**：支持最多一次、最少一次。
  - **消费模型**：一个Queue上的一条消息只能被一个Consumer消费。 通过Exchange将消息投递到多个Queue上可以实现一条消息被多次消费。

存储模型
* Kafka
  - 每个Partition一个目录，目录中包含多个append log分段文件。
* RocketMQ
  - 所有Queue都保存在一个顺序写的commit log上。
  - 默认每10秒消息会被定期清理。
* RabbitMQ
  - Queue和消息可以单独配置是否持久化。


## 最佳实践

通用方案
  - 数据同步服务：消息实时推，然后通过定时拉取进行兜底，防止中间件宕机。
  - 如何分配主题：
    + 有顺序要求的事件应放在同一主题且分区键相同
    + 系统中一个实体另一个实体或两个实体常放在一起使用也应该放在同一主题
    + 消费者常常订阅一组特定的主题时应该将这些主题合并

Kafka
* broker.id设为IP最后一位
* log.dirs所有broker设为一样
* num.partions将partions的备份个数默认设为1
* 历史消息保留不超过1个月
* 分区数和节点数相近，replica为3
* 队列按`系统名_业务名_操作名`命名
* group.id单机部署采用应用名，集群部署采用应用名+编号，不同应用消费同一topic则设为topic名
* offset不自动提交，由Kafka管理offset
* 可通过增加分区提高性能。

RocketMQ
* 同步刷盘：flushDiskType=SYNC_FLUSH
* 可靠性投递至少发送到2个以上节点
* 通过tag在broker端实现消息过滤






