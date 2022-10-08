---
weight: 30
title: 配置管理
bookCollapseSection: true
---

# 配置管理

## 知识点

Apollo

- 长轮询+推拉结合：客户端和服务端通过长轮询连接；服务端有新配置会推送给客户端；客户端也会定时进行拉取操作避免遗漏；
- 多级缓存：客户端会将配置文件在内存和本地磁盘上各保存一份；每次获取配置项时优先从内存取；
- 作用：动态配置，功能开关

## 最佳实践

### Apollo

1. 本地配置文件按 `application.yml`、`application-db.yml` 等来进行功能分组，其中db等后缀对应Apollo上的namespace。
2. 在Apollo上创建应用后，将app.id配置在`application.yml`中。
3. 在Apollo上创建namespace进行分组配置以及配置的继承。
4. 将ConfigServer部署为集群。

## SpringBoot

bootstrap.yml

```yml
spring:
  profiles:
    active: dev
```

该配置会自动启用`application-dev.yml`文件，也可以在运行项目时通过命令 `-Dspring.profiles.active=dev ` 启用。











