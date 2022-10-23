---
weight: 3
title: RabbitMQ
---

# RabbitMQ

## 基本概念

### 工作模式

- 工作队列模型（Work queues）：默认，路由为队列名，多个消费者消费同一队列时，消息轮流分配。
- 发布订阅模型（Publish/Subscribe）：路由为exchanger，exchanger发布消息到绑定的队列上，每个队列全量消息，多个消费者消费同一队列时，消息轮流分配。
- 路由模型（Routing）：路由为exchanger，exchanger发布消息到指定routingkey的队列上，routingkey采用完全匹配。
- 主题模型（Topics）：路由为exchanger，exchanger发布消息到指定topic的队列上，topic是通配符。
- RPC模式：基于Direct Exchange使用MQ实现RPC异步调用。

### Exchange Type

- direct：默认。按routingKey发送到匹配的队列。一个队列可以绑定多个key，多个队列可以有相同的key，在队列层面这些key被称为bindingKey。
- topic： 根据topic条件模糊匹配发送到指定队列。这种routingKey是使用点（.）分隔的多个单词，单词可以使用通配符，`*`表示正好一个单词，`#`表示0个或多个单词。
- headers
- fanout：广播消息到所有队列。

### 节点类型

分类
- RAM：数据保存在内存中，可提高性能。
- Disk：数据保存在磁盘上。集群中至少有一个RabbitMQ节点需要为Disk类型，用于保存元信息。

搭建生产环境时可以通过设置3台机器，其中1台为Disk，2台为内存。这种方式可以提高系统吞吐量，但是一旦Disk宕机后就不支持写入操作了。

## 集群架构

### 集群模式

普通集群

建立多个RabbitMQ实例。此时queue还是只保存在创建其的机器上，但是其它机器会保存元数据。如果消费者连接上了其它节点，则该节点会根据元数据的信息将数据从存有该数据的节点上拉取过来。

镜像集群

包含镜像队列的普通集群。镜像队列会保存到多个节点上。每次写消息时，多个实例间会进行同步。缺点是开销大，每个节点都有全量的queue。创建队列的节点为主节点，备份的节点为镜像节点。

仲裁集群

3.8版本后推出。底层采用Raft协议确保主从的数据一致性，用于替代镜像模式。

