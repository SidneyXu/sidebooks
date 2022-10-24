---
weight: 4
title: 日志管理
---

# 日志管理

## 知识点

建设中。。。

## 最佳实践

IDEA

- Grep Console Plugin：按日志级别对控制台的输出结果进行染色。

## 分布式系统的日志收集

ELK + Filebeat

- Filebeat：收集日志。需集成在每个需要收集日志的服务器上。
- Logstash：日志解析和传输。接受Filebeat的数据，经过处理后发送给ES。如果数据量很大，就需要部署两种Logstash，一种负责解析，一种负责发送给消息队列，因为Filebeat无法直接和消息队列通信。
- ElasticSearch：日志存储。
- Kibana：日志展示。有能力应按需自建日志大盘。

架构图

Log File -> Filebeat -> Logstash -> ES -> Kibana

或

Log File -> Filebeat -> Logstash -> Kafka -> Logstash -> ES -> Kibana
