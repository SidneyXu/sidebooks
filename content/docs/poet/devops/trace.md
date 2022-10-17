---
weight: 5
title: 调用链监控
---

# 调用链监控

## 知识点

核心概念
- Trace：一次调用链。
- Span：调用链上的一次局部方法调用。

Cat
- 侵入式，需编写代码实现调用链传递。
- 客户端内置缓冲池，超过缓冲池大小的数据会被丢弃。

Skywalking
- 可以通过Agent实现自动探针。
- 没有缓冲池，无法及时发送的数据会被直接丢弃。

## 最佳实践

建设中

## Cat

### 报表一览

#### 应用报错大盘

![alt](/images/devops/cat1.png)

#### 业务大盘

基于埋点，大部分情况下还是推荐专门的Metrics监控。

![alt](/images/devops/cat2.png)

### LogView

#### 普通LogView

![alt](/images/devops/cat3.png)

#### 分布式LogView

![alt](/images/devops/cat4.png)

#### 可视化LogView

![alt](/images/devops/cat5.png)

### 应用报表APM

#### Transaction报表

统计代码的运行时间和调用次数。由于Cat是抽样调查的，所以这里的QPS并不能反应真实情况。

![alt](/images/devops/cat6.png)

#### Event报表

事件产生的次数和分布，比如异常。

![alt](/images/devops/cat7.png)

#### Problem报表

根据Transaction和Event分析出来的异常，包括访问较慢的程序等。

![alt](/images/devops/cat8.png)

#### Heartbeat报表

JVM运行时的内部情况，Load/Memory/GC/Thread等。

![alt](/images/devops/cat9.png)

#### Cache报表

分析缓存命中率。

![alt](/images/devops/cat10.png)

## Skywalking

### APM

Global

![alt](/images/devops/skywalking1.png)

Service

![alt](/images/devops/skywalking2.png)

Instance

![alt](/images/devops/skywalking3.png)

Endpoint

![alt](/images/devops/skywalking4.png)

### Database

![alt](/images/devops/skywalking5.png)

### 拓扑图

![alt](/images/devops/skywalking6.png)

### Tracing

![alt](/images/devops/skywalking7.png)
