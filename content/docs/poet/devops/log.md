---
weight: 3
title: 日志管理
---

# 日志管理

## 知识点

建设中

## 最佳实践

IDEA

- 安装Grep Console插件，使控制台的日志按颜色输出。

## 分布式系统的日志收集

方案：ELK + Filebeat

- Filebeat：收集日志。安装在每个需要收集日志的服务器上。
- Logstash：日志解析。接受Filebeat的数据，经过处理后发送给ES。如果数据量很大，则需要部署两种Logstash，一种负责解析，一种负责发送给消息队列，因为Filebeat目前不具备和消息队列通信的能力。
- ElasticSearch：日志存储。
- Kibana：日志展示。更推荐按需自制大盘。

架构图

Log File -> Filebeat -> Logstash -> ES -> Kibana

或

Log File -> Filebeat -> Logstash -> Kafka -> Logstash -> ES -> Kibana
