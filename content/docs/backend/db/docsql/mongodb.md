---
weight: 1
title: MongoDB
---

# MongoDB

## 集群架构

- 副本集
	+ 最小副本集：1主 1从 1仲裁。
	+ 每个副本集允许挂掉一个节点。
- 分片集群
	+ 最小分片集群：三个shared，每个shared是一个1主1从的复制集；3个configserver；3个mongos。
	+ 每个分片节点相当于一个副本集，允许挂掉一个节点。
	+ 任何分片节点都不能挂掉。
	+ configserver挂掉后无法做数据迁移和自动重平衡等操作，但集群仍然可以读写。
	+ mongos路由节点允许挂掉2个。