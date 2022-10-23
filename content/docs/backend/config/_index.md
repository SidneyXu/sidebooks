---
weight: 30
title: 配置管理
bookCollapseSection: true
---

# 配置管理

## 知识点

Apollo

- 配置同步：采用长轮询+推拉结合。Apollo客户端和服务端通过长轮询维持连接；服务端有新配置会推送给客户端，客户端也会定时执行拉取避免遗漏。
- 多级缓存：客户端会将配置文件在内存和本地磁盘上各保存一份；每次读取配置项时优先从内存中进行获取。
- 作用：动态配置，功能开关。

## 最佳实践

### Apollo

1. 本地配置文件按 `application.yml`、`application-db.yml` 等命名方式进行功能分组，其中db等后缀对应着Apollo上配置的`namespace`。
2. 在Apollo上创建应用后，将应用ID配置在程序中的`application.yml`文件的`app.id`。
3. 在Apollo上创建`namespace`进行分组配置，利用配置继承减少配置项。
4. 将ConfigServer部署为集群实现HA。

### SpringBoot

#### 环境配置

- 通过配置文件指定环境

  ```yml
  spring:
    profiles:
      active: dev
  ```
  
  以上配置会自动启用`application-dev.yml`文件。需要通过Apollo在不同环境下更新该配置。

- 通过运行脚本指定环境

  在项目运行时添加启动项 `-Dspring.profiles.active=dev` 来指定环境。这样每个环境无需改动配置文件，只需拥有不同启动脚本即可。











