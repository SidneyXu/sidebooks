---
weight: 1
title: Hystrix
---

# Hystrix

## 原理

### Fall back方式

- fail fast：错误时返回异常。
- fail silent：错误时返回空数据。
- static fallback：错误时返回缺省值。
- fallback via network：错误时调用备用服务。
- primary+secondary with fallback：新功能上线在secondary，先走老功能primary。

### 隔离方式

- 线程池隔离：每种任务采用独立的线程池执行。支持异步调用、排队、超时等特性。适合线程数量可控以及远程调用第三方服务时。如服务间调用，数据库访问。
- 信号量隔离：每种任务使用信号量限制最大并发量。不支持异步调用、排队、超时等特性。适合fanout场景和高性能的内部应用。如网关和缓存。

### 断路器原理

基于滚筒式统计，默认1s一个桶，一个统计周期10个桶（10s）。Hystrix会统计每个桶内调用成功和失败次数来决定是否打开断路器。一旦打开后每5s进入半开状态放过一个请求，如果该请求成功返回则关闭断路器。

## 使用方法

集成在所有对外发起调用的地方。可以配置在类上进行统一降级，直接返回定制的错误或”服务繁忙，请稍候再试”等信息。

## DashBoard

集成Hystrix的客户端应用通过提供stream供Hystrix DashBoard进行展示。

服务大盘

![image](/images/tolerance/turbine3.png)

指标说明

![image](/images/tolerance/turbine4.png)


## Turbine

用于汇总多个Hystrix客户端提供的stream信息。

对接Eureka

![image](/images/tolerance/turbine1.png)

网关集成Hystrix

![image](/images/tolerance/turbine2.png)


