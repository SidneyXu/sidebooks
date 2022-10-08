---
weight: 20
title: Web服务器
bookCollapseSection: true
---

# Web服务器

## 知识点

### Tomcat

- 何时修改连接数
  - CPU利用率过低，QPS上不去

## 最佳实践

Tomcat配置

```yml
# NIO
## 最大连接数，超过的请求会进入队列，默认值为10000，应大于acceptCount+maxThreads。
max-connection: 8192
## 最大线程数，默认值200。
max-threads: 150-200
## 等待队列，默认值100。
accept-count: 1000
## 核心线程数，默认值10。
min-spare-threads: 50-100
## 连接超时时间，默认值20。
connection-timeout: 10s
```

## Tomcat



## Jetty

