---
weight: 1
title: Apollo
---

## Apollo

应用通过`app.id`绑定Apollo上的配置。通过Apollo服务器按指定namespace覆写对应的`application.yml`文件。

### 原理

#### 服务端

![apollo](/images/config/apollo1.jpg)

Config Server一般配置多个，客户端通过 Eureka 服务发现连接该配置集群。为了解决其它语言没有Eureka的问题，Apollo引入了 MetaService，添加了一层代理层来完成服务发现。

当修改数据后Portal会调用AdminService，AdminService会往DB中插入一条消息记录，ConfigService会定时扫描该表，有新记录则通知客户端。服务端和客户端之间使用长轮询维持连接。之所以没有引入消息队列是为了保持架构简单，减少复杂性。

#### 客户端

- 推拉结合：客户端和服务端保持一个长连接，配置实时推送 ；为了避免遗漏，客户端也会定期拉取(fallback)。
- 配置缓存在内存：本地再缓存一份。
- 应用程序：通过Apollo客户端获取最新配置；订阅配置更新通知。

#### 基本概念

1. 应用 application：配置唯一的appid

2. 环境 environment：dev/fat/uat/pro

3. 集群 cluster：一个应用不同实例的分组，主要用于不同的数据中心

4. 命名空间 namespace：一个应用下不同配置的分组，默认分组为application；一般按照功能分组，如数据库配置，消息队列配置，应用配置；也可以引用不属于任何服务的公共配置，如线程池数量等
   1. 命名空间分类：私有，公有（需全局唯一），关联（私有继承公有）
   1. 私有命名空间会覆盖公共命名空间。不同命名空间应用中先定义的先使用。
5. 配置项item：支持kv，json，xml，定位方式如下：
   1. 私有配置env+app+cluster+namespace+item_key 
   2. 公有配置env+cluster+namespace+item_key

6. 权限

### Portal页面

基本页面

![apollo](/images/config/apollo2.png)

通过文本方式编辑配置，方便进行大量修改

![apollo](/images/config/apollo3.png)

可以查看已连接到Apollo的实例

![apollo](/images/config/apollo4.png)



