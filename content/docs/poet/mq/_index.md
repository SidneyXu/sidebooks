---
weight: 100
title: 消息中间件
bookCollapseSection: true
---

# 消息中间件

## 消息队列中间件的对比

- 性能
  - Kafka：几十万级别。通过批量和异步获得高吞吐量，时延也较高。如果每秒数据量没那么多情况下时延会很高，不太适合在线业务场景。在大数据和流计算方面生态优秀。允许n-1个节点失败。
  - RocketMQ：几十万级别。时延低。
  - RabbitMQ：十万级别。消息堆积会引起性能下降。
- 基本架构
  - Kafka：Producer、Consumer、Consumer Group、Broker、一个Topic多个Partition
  - RocketMQ：Producer、Consumer、Consumer Group、一个Topic多个Queue
  - RabbitMQ：Exchange、Channel、Producer、Consumer、Queue
- 推拉模式
  - Kafka：基于长轮询的拉模式。
  - RocketMQ：支持推拉，但拉模式也是基于长轮询。
  - RabbitMQ：支持推拉模式。
- 消费者数量
  - Kafka：线程数量和partition数量保持一致，一个partition只能被同组的一个消费者消费。
  - RocketMQ：同Kafka。
- 消费语义
  - Kafka：最多一次，最少一次（推荐），正好一次。正好一次是通过transaction幂等使broker确保同样消息只落地一次，消费端不保证。
  - RabbitMQ：最少一次，最多一次。
- 一条消息被多个消费者消费
  - Kafka：多个consumer group消费同一个partition。Consumer Group中的consumer是竞争关系。
  - RocketMQ：多个consumer group消费同一queue。Consumer Group中的consumer是竞争关系。
  - RabbitMQ：每个消费者一个Queue，通过Exchange将一条消息投递到多个Queue中。消费时不指定队列，而是指定Exchange。
- 消息激增扩容
  - Kafka：增加partition数量和consumer数量。旧partition里的消息不会重新分配，新partition只会新消息有用。
- 消息何时被删除
  - Kafka：基于过期时间删除消息，没有过期时无论是否被消费都会被保存。
  - RocketMQ:不会被立即删除。
- 消息失败重试
  - Kafka：不支持重试
  - RocketMQ：支持定时重试
- 消息顺序
  - Kafka：partition有序，但Broker宕机后就会产生乱序。
  - RocketMQ：queue有序，支持严格顺序，一台Broker宕机后发送消息会失败，但不会乱序。
- 消息回溯
  - Kafka：按Offset回溯。
  - RocketMQ：按时间回溯（精确到毫秒）
- 事务支持
  - Kafka：支持。提交事务时失败则抛出异常。
  - RocketMQ：支持。提交事务时失败后，中间件可通过反查接口查询事务状态。
  - RabbitMQ：支持。
- Topic和队列设置数量
  - Kafka：分区数量=Topic的吞吐量/Consumer的吞吐量。单机上一个partition一个目录，由多个segement文件组成。读写消息就是从某一个offset开始顺序读写。当partition数量过多时Load就会更高，写入消息的响应时间就会变慢。
  - RocketMQ：一个应用一个Topic，支持通过Tags在broker端实现消息过滤以区分不同功能。单机上每个队列一个文件，不会因队列增大Load发生明显变化。
  - RabbitMQ：由Exchange决定消息如何被投递。
- 水平扩展
  - RabbitMQ：routeingkey作为分片键绑定不同queue。


## 最佳实践

- 通用方案
  - 数据同步服务：消息实时推，然后通过定时拉取进行兜底，防止中间件宕机。

- 如何分配主题
  - 有顺序要求的事件应放在同一主题且分区键相同
  - 系统中一个实体另一个实体或两个实体常放在一起使用也应该放在同一主题
  - 消费者常常订阅一组特定的主题时应该将这些主题合并
- Kafka
  - broker.id设为IP最后一位
  - log.dirs所有broker设为一样
  - num.partions将partions的备份个数默认设为1
  - 历史消息保留不超过1个月
  - 分区数和节点数相近，replica为3
  - 队列按`系统名_业务名_操作名`命名
  - group.id单机部署采用应用名，集群部署采用应用名+编号，不同应用消费同一topic则设为topic名
  - offset不自动提交，由Kafka管理offset
- RocketMQ
  - 同步刷盘：flushDiskType=SYNC_FLUSH
  - 可靠性投递至少发送到2个以上节点
  - 通过tag在broker端实现消息过滤


## 基础知识

### 1、消费模型

- 队列模型：同一队列上的消费者处于竞争状态。一条消息只能被一个消费者消费。RabbitMQ采用该模型。
- 发布-订阅模型：基于主题订阅，一条消息可以被多个消费者消费。大部分消息中间件采用该模型。

### 2、分区模型

Kafka的partition或RocketMQ的Queue主要为了提高吞吐量实现并行消费，进行水平扩展。此外也可以解决手动提交时因为前面的消息没有被消费后面的消息一直都不能被消费的情况。

## RabbitMQ





