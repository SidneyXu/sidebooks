---
weight: 1
title: Spring Cloud Gateway
---

## Spring Cloud Gateway

### 原理

基于Netty实现，通过过滤器实现网关功能。

![gateway](/images/gateway/gateway.png)

- 过滤器分为全局过滤器和网关过滤器。所有过滤器构成一个chain。全局过滤器应用到所有路由中。每个过滤器都可以通过代码实现类似Zuul的Pre，Post，Route过滤器的功能。
- FilterFactory用于按配置生产Filter，通过自定义FilterFactory可以在配置文件中按需配置自定义Filter。


