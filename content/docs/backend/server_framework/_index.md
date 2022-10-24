---
weight: 70
title: 服务框架
bookCollapseSection: true
---

# 服务框架

## 知识点

### Dubbo

- 负载均衡策略：随机（默认）、轮询、最少活跃、一致性Hash。
- 集群容错：快速失败、自动切换节点、忽略失败、失败自动回复、并行广播（一个成功）、逐个广播（全部成功）
- 支持协议：dubbo（默认）、rmi、webservice、http、hessian(http)、http表单提交、memcache、redis

## 最佳实践

Dubbo
- dispatcher：all或message。协议派发方式，默认为all。
- threadpool：fix。线程类型，默认为fix。
- threads：150-200。业务线程池大小，默认为200。线程数不足时优先考虑将dispatcher改为message。
- iothreads：I/O线程池大小。默认为CPU核心数+1。
- queues：等待队列，默认为0。
- protocol：
  + 高并发小数据量：dubbo
  + 高并发大数据量：hessian


