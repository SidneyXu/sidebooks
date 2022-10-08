---
title: 负载均衡与反向代理
weight: 10
bookCollapseSection: true
---

# 负载均衡与反向代理

## 知识点

- 如何实现负载均衡：
  - DNS实现地理级别负载均衡；硬件（F5）实现集群层面负载均衡；软件（Nginx/OpenResty、LVS）实现机器级别负载均衡。
- 为什么不能使用DNS轮询实现负载均衡
  - 旧浏览器在访问一个IP失败后就会停止访问下一个IP地址
- 如何实现高可用
  - 双机主备模式：DNS绑一个VIP + 双keepalived/Nginx。一台Nginx通过VIP提供服务，另一台仅作为备机。
  - 双机主主模式：DNS绑两个VIP + 双keepalived/Nginx。两台Nginx通过两个VIP同时提供服务，互相作为备机。
- 如何添加新机器
- 何时需要添加新机器
- 如何监视Nginx的状态

## 最佳实践



## 负载均衡

### DNS 负载均衡

用于实现地理位置级别的均衡，如北京用户访问北京机房。原理是 DNS 解析同一个域名可以访问不同 IP 地址。

### 硬件负载均衡

F5和 A10

### 软件负载均衡

Nginx 和 LVS

- Nginx 是软件的7层负载均衡，支持 HTTP，Email 协议
- LVS 是 Linux 内核的4层负载均衡，和协议无关

Nginx 的底层模块一般都是用 C 语言写的，如果我们想在 Nginx 的基础之上写业务逻辑，还得借助 OpenResty。

OpenResty 是一个基于 Nginx 与 Lua 的高性能 Web 平台，它使我们具备在 Nginx 上使用 Lua 语言来开发业务逻辑的能力。

## 反向代理








