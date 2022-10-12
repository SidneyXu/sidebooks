---
title: 负载均衡
weight: 10
bookCollapseSection: true
---

# 负载均衡与反向代理

## 知识点

### 通用

- 负载均衡
  + 为什么不能使用DNS轮询实现负载均衡
    + 部分旧浏览器在访问一个IP失败后就会停止访问下一个IP地址。

### Nginx
  + 负载均衡策略：轮询、加权轮询、ip_hash。

## 最佳实践

- 负载均衡最佳实践
  - DNS层面实现地理级别负载均衡；
  - 硬件（F5）层面实现集群层面负载均衡；
  - 软件（Nginx/OpenResty、LVS）层面实现机器级别负载均衡。
- Nginx高可用
  - 双机主备模式：DNS绑一个VIP + 双keepalived/Nginx。核心是一台Nginx通过VIP提供服务，另一台Nginx仅作为备机。
  - 双机主主模式：DNS绑两个VIP + 双keepalived/Nginx。核心是两台Nginx通过两个VIP同时提供服务，并且两台Nginx互为备机。

## 负载均衡

### DNS层面负载均衡

用于实现地理位置级别的均衡，如北京用户访问北京机房。原理是 DNS 解析同一个域名可以获得不同的 IP 地址。

### 硬件层面负载均衡

F5 和 A10

### 软件层面负载均衡

Nginx 和 LVS

- Nginx 为软件层面的7层负载均衡，支持 HTTP，Email 协议。
  + OpenResty相当于支持Lua脚本的Nginx，可以通过Lua脚本为Nginx添加业务逻辑。
- LVS 是 Linux 内核的4层负载均衡，和协议无关。









