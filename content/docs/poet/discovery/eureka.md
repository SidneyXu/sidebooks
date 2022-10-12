---
weight: 1
title: Eureka
---

# Eureka

## 基本概念

### 原理

Eureka Client定期从Eureka Server获取配置保存到缓存中。应用调用其它服务时会从本地缓存中获取可用的服务器。

### 基本组成

- Eureka Client：集成在客户端中，用于进行服务注册和服务发现，Client既可以是Service Provider也可以是Service Consumer。Client会优先访问位于同一个Zone里的Service Consumer。
- Eureka Server：Eureka 服务端，提供页面查看功能，搭建多个服务端集群时需要将自己也注册到Eureka Server上。
- 自我保护机制：开启配置后，如果Eureka Server无法和一定数量Eureka Client进行连接时，停止新服务注册和服务踢出功能，待连接恢复正常，主要为了防止网络抖动将正常服务踢出。

![eureka](/images/discovery/eureka.jpg)

## 基本使用

### 管理页面


![eureka](/images/discovery/eureka1.png)

### with Ribbon

Ribbon可以实现客户端负载均衡，原理如下：
1. Ribbon拦截器拦截RestTemplate的请求。
2. 根据用户指定的策略，从本地保存的Eureka服务注册列表中选择一个Server地址替换请求中的服务名。其中 Ribbon 提供了多种客户端负载均衡策略，例如轮询、随机、根据响应时间加权等。