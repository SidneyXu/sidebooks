---
weight: 60
title: 服务容错
bookCollapseSection: true
---

# 服务容错

## 知识点

容错模式

- 超时：限制请求的最大处理时间。
- 限流：限制请求的最大并发数。
- 熔断：当请求返回的错误数到达阀值时进行自动降级。
- 隔离：隔离不同的依赖。
- 降级：服务不可用时进行特殊处理。熔断可以看做是降级的一种实现方式。

大部分容错机制都可以直接可以在网关层解决。

## 最佳实践

- 大盘：有指标监控服务的优先使用指标监控服务，没有监控服务的再使用各容错框架内部提供的Dashboard。
- 限流
	+ Nginx层：Nginx 中包含了两个限流模块：ngx_http_limit_conn_module 和 ngx_http_limit_req_module，前者是用于限制单个 IP 单位时间内的请求数量，后者是用来限制单位时间内所有 IP 的请求数量。
	+ 应用层：Zuul RateLimit 或 Guava RateLimiter。



