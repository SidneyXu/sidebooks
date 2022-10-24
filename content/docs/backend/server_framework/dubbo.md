---
weight: 1
title: Dubbo
---

# Dubbo

官网：[https://dubbo.apache.org/zh/](https://dubbo.apache.org/zh/)

## 原理

### 总体架构

* Container：服务运行容器，默认为Jetty，负责运行Provider。
* Provider：服务的提供方，启动后会向Registry进行注册。
* Consumer：服务消费方，启动后向Registry获取订阅列表并缓存到本地，Registry也会基于长连接定时推送变更。
* Register：注册中心，可选值为：Zookeeper、Nacos、Consul、multicast、redis或simple。
* Monitor：统计服务调用次数和时间的监控中心，Provider和Consumer会在内存中进行统计后每分钟发送给Monitor。

### Dispatcher

- all：所有消息都派发到线程池。
- direct：所有消息都在I/O线程上直接执行。
- message：只有响应消息派发到线程池，其它连接、心跳等事件直接在I/O线程上执行。
- execution：只有请求消息派发到线程池，其它响应、连接、心跳等事件直接在I/O线程上执行。
- connection：在I/O线程上，将连接断开事件放入队列，其它消息派发到线程池。