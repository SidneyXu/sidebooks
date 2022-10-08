---
weight: 60
title: 服务容错
bookCollapseSection: true
---

# 服务容错

## 知识点

容错模式

- 超时：限制请求的最大处理时间。
- 限流：限制最大并发数。
- 熔断：错误数到达阀值时进行特殊处理。
- 隔离：隔离不同依赖调用。
- 降级：服务不可用时降级，熔断是降级的一种实现方式。

80%的容错都可以直接可以在网关层解决。

## 最佳实践

Nginx层

- Nginx 中包含了两个限流模块：ngx_http_limit_conn_module 和 ngx_http_limit_req_module，前者是用于限制单个 IP 单位时间内的请求数量，后者是用来限制单位时间内所有 IP 的请求数量。

应用层

- Zuul RateLimit 或 Guava RateLimiter




## Sentinel

得物配置的熔断规则

![图片](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74DicxEpwtEX1UrBMld6XA3nHrp3f3oRiauwX9CJ868hIK7kNUYFCXL5iberyd5lz7cd5jFWwjdxxrWmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Sentinel的慢调用比例熔断规则统计的时候，不是等到滑动窗口结束了再去根据这一整个窗口的数据来做判断，而是每次请求都会做判断。

比如拿最上面的配置规则来做例子的话，如果当前窗口的刚开始的前几个请求中（大于5）慢调用比例刚好超过了50%，那么就会触发熔断，断路器直接打开，3s内的所有请求都走降级，然后3s后断路器进入半开状态，如果下一个请求正常了，那么断路器就关闭。 

## resilience4j

