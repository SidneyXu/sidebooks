---
weight: 1
title: Apollo
---

# Apollo

## 基本概念

### 原理

![apollo](/images/config/apollo1.jpg)

Config Server一般配置为集群，客户端通过 Eureka 的服务发现机制连接该集群。为了解决其它语言没有Eureka的问题，Apollo引入了 Meta Server，通过添加代理层来实现服务发现。

当用户在页面上修改配置后，Portal页面会调用Admin Server，Admin Server会往DB中插入一条消息记录，Config Server会定时扫描该表，有新记录通知客户端。服务端和客户端之间使用长轮询维持连接，之所以没有引入消息队列是为了保持架构简单，减少复杂性。


### 基本组成

- 应用 application：唯一性基于appid
- 环境 environment：可选值：dev/fat/uat/pro
- 集群 cluster：将单应用的不同实例进行分组，主要用于不同的数据中心
- 命名空间 namespace：将单应用的配置进行分组，默认分组为application。一般可以按照功能进行分组，例如数据库配置，消息队列配置，应用配置等；也可以用来引用不属于任何服务的公共配置，如线程池数量等
   +  命名空间分类：私有，公有（需全局唯一），关联（私有继承公有）
   +  私有命名空间会覆盖公共命名空间。
   +  不同命名空间应用中先定义的先使用。
-  配置项item：支持kv，json，xml，定位方式如下：
   + 私有配置env+app+cluster+namespace+item_key 
   + 公有配置env+cluster+namespace+item_key
- 权限

## Portal页面

![apollo](/images/config/apollo2.png)

大量修改配置时，可以选择使用【文本】Tab进行编辑。

![apollo](/images/config/apollo3.png)

【实例列表】Tab上可以查看当前已连接到Apollo的所有客户端实例。

![apollo](/images/config/apollo4.png)



