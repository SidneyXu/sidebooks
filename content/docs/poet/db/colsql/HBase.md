---
weight: 2
title: HBase
---

# HBase

## 基本概念

- HBase客户端
	+ 与 HMaster 通信进行管理类的操作。
	+ 与 HRegionserver 通信进行读写类操作。
- Zookeeper
	+ 保证任何时候，集群中只有一个 Master。
	+ 存储所有 Region 的寻址入口。
	+ 实时监控 HRegionServer 的上线和下线信息，通知 Master。
	+ 存储 HBase 的 schema 和 table 元数据。
- Master 节点
	+ 管理 HRegionServer 的负载均衡，调整 Region 分布。
	+ 在 Region Split 后，负责新 Region 的分配。
	+ 在 HRegionServer 停机后，负责失效 HRegionServer 上的 Region 迁移
	+ HMaster 失效仅会导致所有元数据无法被修改，但是表数据还是可以正常读写。
- Region 节点
	+ 存储数据，负责读写操作。