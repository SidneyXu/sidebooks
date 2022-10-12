---
weight: 30
title: 配置管理
bookCollapseSection: true
---

# 配置管理

## 知识点

Apollo

- 长轮询+推拉结合同步配置：Apollo客户端和服务端通过长轮询维持连接；服务端有新配置会推送给客户端，客户端也会定时执行拉取避免遗漏；
- 多级缓存：客户端会将配置文件在内存和本地磁盘上各保存一份；每次读取配置项时优先从内存中进行获取；
- 作用：动态配置，功能开关；

## 最佳实践

### Apollo

1. 本地配置文件按 `application.yml`、`application-db.yml` 等命名方式进行功能分组，其中db等后缀对应着Apollo上配置的namespace。
2. 在Apollo上创建应用后，将应用ID配置在程序中的`application.yml`文件的`app.id`项上。
3. 在Apollo上创建namespace进行分组配置，可以利用配置继承减少配置项。
4. 将ConfigServer部署为集群模型实现高可用。

### SpringBoot

bootstrap.yml

```yml
spring:
  profiles:
    active: dev
```

该配置会自动启用`application-dev.yml`文件，更推荐项目运行时通过命令 `-Dspring.profiles.active=dev ` 启用该配置。这样每个环境无需改动设置，只需拥有不同项目启动脚本即可。











