---
weight: 40
title: 服务发现
bookCollapseSection: true
---

# 服务发现

## 知识点

Eureka

- 续约：Eureka Client每30秒会发出心跳进行续约，更新注册表。由于是定时拉取，所以无法保证实时性。
- 失效剔除：Eureka Server每隔60秒会将最近90秒内没有续约的服务剔除。
- 自我保护：当15分钟内Eureka Server发现心跳失败比例低于85%时（即无法和一定数量的Client通信时），会暂时停止剔除失效服务，且新服务的注册不会被同步到其它节点。

## 最佳实践

- Eureka
  - HA：多个Eureka Server互相注册，同机房的Eureka Server属于同一个Zone。
  - 如何应对Eureka无法及时将下线服务剔除
    - 使用Ribbon时，设置MaxAutoRetriesNextServer=1，让Ribbon在调用失败时自动尝试下一个节点。
    - 调整Eureka的拉取频率。





