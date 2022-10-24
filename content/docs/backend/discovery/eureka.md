---
weight: 1
title: Eureka
---

# Eureka

## 基本概念

### 原理

Eureka Client定期从Eureka Server获取配置保存到缓存中。应用调用其它服务时会从本地缓存中获取可用的服务器。

### 基本组成

- Eureka Client：集成在应用中，用于进行服务注册和客户端的服务发现，Client既可以是Service Provider也可以是Service Consumer，会优先访问位于同一个Zone里的Service Provider。
- Eureka Server：Eureka 服务端，提供页面查看功能，搭建多个服务端集群时需要将自己也注册到Eureka Server上。同一机房的Eureka Server一般都注册在同一Zone中。

![eureka](/images/discovery/eureka.jpg)

## 基本使用

### 管理页面

![eureka](/images/discovery/eureka1.png)

### Ribbon

Eureka+Ribbon可以实现客户端负载均衡，原理如下：
1. Ribbon拦截器拦截所有标有@LoadBalanced注解的RestTemplate对象发起的请求。
2. 根据用户指定的策略（轮询、随机、根据响应时间加权等），Ribbon从本地保存的Eureka服务注册列表中选择一个Server地址替换请求地址中的服务名。