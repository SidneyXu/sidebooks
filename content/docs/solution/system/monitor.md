---
weight: 2
title: "监控大盘"
---

# 监控大盘

## 设计方案

市面上监控系统虽多，但功能太多且碎片化，查找问题需要大量沟通，定位问题繁琐。

自定义监控系统实现方案

- 通过Agent进行监控数据采集，Agent是一个独立服务。
- Agent定时（3s）采集后将数据同步到Monitor。
- Monitor保存监测数据。
- Dashboard展示数据，通过红黄绿颜色表示节点状态为不健康、未知/警告、健康，同时显示出错信息。
- Dashboard界面应该可以定制化。
- 各节点的配置信息可以保存在单独的注册中心，也可以直接保存在Monitor中，由Agent定期获取最新配置。

## 数据建模

表结构
- 系统信息表：系统名称，页面布局方式。
- 节点信息表：节点名称、节点类型、节点机器、节点账号、节点所属系统、节点所属应用、告警邮箱、告警收集。
- 节点监控状态表：节点名称、节点类型、错误信息、上报时间、调用数、错误数、平均响应时间。

## Agent设计方案

- Redis
	+ Agent通过Jedis尝试对Redis进行读写操作，操作正常则表明Redis是健康的。
- MQ
	+ Agent通过MQ API检测 MQ 是否有活跃的消费者在连接，同时检测队列积压的消息数量。如果没有活跃的消费者，或者未消费的消息超过阀值，就表明当前的 MQ 节点是不健康的。
- Database
	+ Agent通过JDBC连接数据，尝试读写操作，正常返回则表明Redis是健康的。
- Web Server
	+ 每个服务集成SDK，统计采集窗口内（3s）的接口调用次数、出错次数、RT等额外信息。
	+ Agent通过网络请求调用服务中集成的SDK，获取健康信息和额外信息。

{{< hint info >}}
不少中间件在空间不足或网络分区等情况下仍然可以接受读取操作，但这种情况并不表示节点是健康的，所以对于写操作的监测是必不可少的。
{{< /hint >}}

## Web Server SDK设计方案

- 每个时间窗口一个HashMap，key=接口名，value=调用详情，包含调用次数、出错次数、RT等。
- Agent调用SDK时，SDK统计健康状态。如果没有错误产生，则节点健康；错误次数小于一定阀值则为警告；错误次数大于一定阀值则为错误。
- SDK统计RT健康状态时，不能采用预先设定的RT阀值，而应该根据节点运行轨迹动态计算出正常请求的AVG RT。
- 一个应用内通常包含多个接口，SDK反应的节点健康状态是所有接口中最差的。



