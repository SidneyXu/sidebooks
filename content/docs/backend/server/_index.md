---
weight: 20
title: Web服务器
bookCollapseSection: false
---

# Web服务器

## 知识点

### 通用

- 常见服务器
  + Tomcat
  + Jetty
  + undertow


### Tomcat

- 何时应增加连接数/线程数
  - QPS上不去，而CPU利用率过低。

## 最佳实践

### Tomcat

Tomcat仅承担服务器的功能，单Tomcat单服务，虚拟主机功能由Nginx提供。

常用配置项

```
# NIO
## 最大连接数，超过的请求会进入队列，默认值为10000，该值应大于acceptCount+maxThreads。
max-connection: 8192
## 最大线程数，默认值200。QPS上不去时可增加到800。
max-threads: 150-200
## 等待队列，默认值100。
accept-count: 1000
## 核心线程数，默认值10。
min-spare-threads: 50-100
## 连接超时时间，默认值20。
connection-timeout: 10s
```

