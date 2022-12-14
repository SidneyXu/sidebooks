---
weight: 11
title: 客服系统
---

# 客服系统

## 客服分配算法

业务要求

- 优先机器人处理，不行再转人工；具体策略可按时间段调整，比如说非工作时间只有机器人。
- 上次服务过的客服优先分配。
- 为该客户服务次数最多的客服优先分配。
- 客服存在最大接待客户数的上限。
- 多种分配策略：空闲率分配、轮询分配、权重分配。

实现方式

1. 按负责的业务模块，将所有客服进行分组。
2. 在Redis中使用Hash存储在线客服的动态属性，如当前状态、正在接待人数、最大接待人数等。
2. 在Redis中使用SortedSet存储不同分组的客服摘要信息，以上次接待时间作为score，最新接待客户的客服移动到SortedSet最后。
2. 客户提出问题，AI识别后弹出相关FAQ。
2. FAQ无法解决问题时，客户选择业务模块转入人工流程。
4. 用户接入时，查找是否存在上次接待过的客服，具体关系可保存在用户信息中。
	- 存在。从Hash中取出客服信息，将当前接待人数+1，更新SortedSet中的score。
	- 不存在。从SortedSet中取出第一个客服，使用CAS来防止并发竞争。如果失败则通过Shuffle从SortedSet中随机取出一个客服。将当前接待人数+1，更新SortedSet中的score。
6. 客服也可以手动选择客户或者转让客户。
6. 当不存在可分配客服时，客户进入等待池排队。