---
weight: 1
title: Hystrix
---

## Hystrix

### 使用方式

所有对外发起调用的地方，client配置在类上进行降级，直接返回错误或统一的服务繁忙，请稍候再试

### 断路器原理

通过断路器实现，基于滚筒式统计，每秒一个桶，默认10个桶（10s）一个统计周期，统计每个桶内调用成功和失败次数来决定是否打开断路器。一旦打开后每5s进入半开状态放过一个请求，请求成功后关闭断路器。

### Fall back方式

- fail fast：错误时返回异常。
- fail silent：错误时返回空数据。
- static fallback：错误时返回缺省值。
- fallback via network：错误时调用备用服务。
- primary+secondary with fallback：新功能上线在secondary，先走老功能primary。

### 隔离方式

- 线程池隔离：每个任务采用独立的线程，支持排队超时等特性。适合线程数量可控以及调用不了解性能的第三方服务。主要用于服务间调用，数据库访问。
- 信号量隔离：相同任务使用同一线程，不支持排队超时等特性。适合高扇出的场景和高性能的内部应用。主要用于网关和缓存。

### Turbine

Turbine用于汇总多个Hystrix客户端提供的stream信息。

### 生产实践

对接Eureka

![image](/images/tolerance/turbine1.png)

网关集成Hystrix

![image](/images/tolerance/turbine2.png)



### DashBoard使用

![image](/images/tolerance/turbine3.png)

![image](/images/tolerance/turbine3.png)


![image](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DicxEpwtEX1UrBMld6XA3nHrp3f3oRiauwX9CJ868hIK7kNUYFCXL5iberyd5lz7cd5jFWwjdxxrWmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)