---
weight: 70
title: 服务框架
bookCollapseSection: true
---

# 服务框架

## 知识点

Dubbo

- 基本组件
  - Container：服务运行容器，默认为Jetty。
  - Provider：服务的提供方，启动后会向Registry进行注册。
  - Consumer：服务消费方，启动后向注册中心获取订阅列表，Registry会定时推送变更。
  - Register：注册中心，短暂宕机不会受到影响。可以使用Zookeeper、multicast、redis或simple。
  - Monitor：定时统计服务调用次数和时间的监控中心。
- 负载均衡策略：随机（默认）、轮询、最少活跃、一致性Hash。
- 集群容错：快速失败、自动切换节点、忽略失败、失败自动回复、并行广播（一个成功）、逐个广播（全部成功）
- 支持协议：dubbo（默认）、rmi、webservice、http、hessian(http)、http表单提交、memcache、redis

## 最佳实践

Http



Dubbo

- dispatcher：all或message。默认all，协议派发方式。
- threadpool：fix。默认，线程池类型。
- threads：150-200。默认200，业务线程池大小。
- iothreads：默认CPU核心数+1，I/O线程池大小。
- queues：默认0，业务线程池配合的等待队列大小。
- 大文件顺畅：hessian协议
- 高并发小数据量用dubbo协议，大数据量用http协议。



## SpringBoot



## Dubbo