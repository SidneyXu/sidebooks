---
weight: 1
title: Eureka
---

## Eureka

### 原理

Eureka Client定期从Eureka Server获取配置保存到缓存中。应用调用其它服务时会从本地缓存中获取可用的服务器。

### 基本概念

- Eureka Client：集成在客户端中，用于进行服务注册和服务发现，Client既可以是Service Provider也可以是Service Consumer。Client会优先访问位于同一个Zone里的Service Consumer。
- Eureka Server：Eureka 服务端，提供页面查看功能，搭建多个服务端集群时需要将自己也注册到Eureka Server上。
- 自我保护机制：开启配置后，如果Eureka Server无法和一定数量Eureka Client进行连接时，停止新服务注册和服务踢出功能，待连接恢复正常，主要为了防止网络抖动将正常服务踢出。
- Ribbon：客户端负载均衡。

### 生产实践

![eureka](/images/discovery/eureka.jpg)

![eureka](/images/discovery/eureka1.png)