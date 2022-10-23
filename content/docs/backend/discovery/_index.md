---
weight: 40
title: 服务发现
bookCollapseSection: true
---

# 服务发现

## 知识点

Eureka

- 续约：Eureka Client每30秒会进行心跳续约，更新注册表，所以无法保证实时性。
- 失效剔除：Eureka Server每隔60秒会剔除最近90秒内没有续约成功的服务。
- 自我保护：当Eureka Server发现15分钟内心跳失败比例低于85%时，会暂停失效剔除。此时虽然仍能接受新服务注册，但新注册的信息不会被同步到其它节点上。

## 最佳实践

- Eureka
  - HA：多个Eureka Server互相注册，同机房的Eureka Server应配置在同一个Zone。
  - 如何应对Eureka无法及时将下线服务剔除
    - 使用Ribbon时，设置MaxAutoRetriesNextServer=1，让Ribbon在调用失败时自动尝试下一个节点。
    - 调整Eureka的拉取频率。





