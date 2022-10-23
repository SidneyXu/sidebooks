---
weight: 2
title: Spring Cloud Gateway
---

# Spring Cloud Gateway

## 原理

![gateway](/images/gateway/gateway.png)

基于Netty实现，通过Filter级联实现网关功能。

- **Predicate**：断言。定义了请求的匹配规则，由哪个Route进行后续处理。
- **Route**：路由。定义了如何对请求进行转发。
- **Filter**：分为Global Gateway Filter和Gateway Filter。Global Gatway Filter应用在所有路由中。每个Filter都可以通过代码实现类似Zuul的Pre、Route、Post的功能。
- **FilterFactory**：用于按配置生产Filter，通过自定义FilterFactory可以在配置文件中配置自定义的Filter。
- **限流**：通过redis-rate-limiter实现基于Redis的令牌桶限流。

