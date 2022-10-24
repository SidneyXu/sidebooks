---
weight: 10
bookFlatSection: true
title: "Server Side"
bookToc: false
---

# Server Side

## 目录

- 负载均衡：Nginx
- Web服务器：Tomcat、Jetty、undertow
- 配置管理：Apollo、SpringBoot Yaml
- 服务发现：Eureka、Consul
- 网关：Zuul、SpringCloud Gateway
- 服务容错：Hystrix、Sentinel、Resilience4j
- 服务框架：
	+ 服务治理：SpringCloud、Dubbo、gPRC
	+ 异步编程：Vert.x、CompletableFuture、RxJava
	+ 服务网格：Istio
- 任务调度：XXL-JOB、Crontab
- 数据库：
	+ 关系型数据库：MySQL
	+ 文档型数据库：MongoDB
	+ 列式数据库：Cassandra、HBase
	+ 全文数据据库：ElasticSearch
	+ 缓存列数据库：Redis、Memcached
	+ 应用内缓存：Caffeine、Ehcache、Guava Cache
	+ 数据库中间件：Sharding-Sphere、MyCat、canal
	+ 消息中间件：Kafka、RocketMQ、RabbitMQ
- 大数据处理：Flink、Spark
- 测试与性能分析：
	+ 压力测试：ab、JMeter、Gatling、locust
	+ 基准测试：JMH
	+ CPU和线程分析：async-profiler
	+ 硬盘和网络分析：
	+ JVM分析：jstat、jmap、GCeasy、GCViewer、jstack
	+ Arthas
	+ chaosblade
- Devops：Linux
	+ CI：Git、Jenkins、Nexus
	+ 日志管理：Filebeat、ELK
	+ 容器编排：k8s、Docker
	+ 调用链监控：Skywalking、Cat
	+ 指标监控：Prometheus、 Grafana

## 常用性能指标参考

- Nginx 负载均衡性能是5万左右。
- Http 请求访问大概在2万左右。
- JVM
	+ YGC 一般在20ms以内。
	+ FGC 一般超过200ms。
- 缓存
  - Memcache 的读取性能在5万左右。
  - Redis 单机读10w，大key较多时则降为4、5万。
- 队列
  - Kafka 几十万到百万级。
  - RocketMQ 几十万级。
  - RabbitMQ 十万级。
- Zookeeper 读写在2万以上。
- MySQL
  - 主键访问在 5ms以内。
  - 4核8G机器上可以支撑 500 的 TPS 和 10000 的 QPS。
- 监控系统
	+ Cat 单机 15W QPS。
- 硬件
	+ SSD磁盘随机读 16us。
	+ 机械硬盘随机读 4ms。



