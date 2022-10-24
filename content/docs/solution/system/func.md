---
weight: 3
title: 基础功能
---

# 基础功能

## 分布式锁

### 设计原则

- 互斥性：同一个时间只能有一个线程持有锁。
- 容错性：即使某一个持有锁的线程异常退出，其他线程仍可获得锁。
- 隔离性：线程只能释放自己的锁，不能释放其他线程的锁。

### 数据库唯一索引

建立唯一索引，加锁时插入数据，释放锁时删除数据。

```
# 加锁
insert into locks values('lock_method_name');
# 释放锁
delete from locks where method_name = 'lock_method_name';
```

- 优点：实现简单；通过DB实现强一致性。
- 缺点：并发度较低；需自行实现超时时间。
- 适合场景：批处理任务防重。

### Redis单节点锁

通过Redis进行加锁，设置超时时间。

```
# Redis层面
set key value [ex <second>|px <mill>] nx
# Java Client层面
template.opsForValue().setIfAbsent(key, value, timeout, TimeUnit.SECONDS);
```

- 优点：实现简单、高效，支持超时时间。
- 缺点：
	+ 由于Redis采用异步复制，可能会出现脑裂，导致多个客户端持有锁；
	+ 超时时间难以预估，可能因FGC等原因造成任务执行中就释放锁，此时也会出现多个客户端持有锁。只能通过将value设为唯一值避免原线程释放了此时已不属于自己的锁。
- 优化：
	+ 设置超时时间时添加一段缓冲时间，也就是说把超时时间设得更长一点。
	+ 在获取锁前先获得版本号，获得锁后执行修改前检查版本号。

### Redission单节点锁（Watchdog）

通过Watchdog机制定期延长加锁时间，可以有效防止FGC等原因造成业务未执行完毕就释放锁的问题。不过对于异步复制引起的脑裂问题仍然没有办法。

- 优点：通过Watchdog确保业务拥有足够时间执行完毕。
- 缺点：
	+ 脑裂引起的多客户端持有锁问题仍然存在。
	+ Watchdog虽然在大部分场合都适用，但FGC还是可能导致锁被提前释放。
- 优化：同单节点锁。

### Redlock集群加锁

尝试在所有Redis节点加锁，并统计一半以上节点的加锁完成时间是否在阀值内，没有超时则加锁成功。

- 优点：多节点加锁以解决脑裂问题。
- 缺点：依赖时间，时钟漂移可能会产生问题；加锁顺序也存在数据安全隐患。目前争议较大，少有实施案例。

### Zookeeper

客户端尝试在Zookeeper上创建临时节点，创建成功的即获得锁。删除节点或断开连接时即释放锁。

- 优点：通过ZK实现强一致性，Leader切换时不提供服务，解决了脑裂问题。
- 缺点：依赖Leader提供服务，并发度低。

## 分布式ID

### 设计原则

- 全局唯一
- 单调递增
- 无序（无法推断规模）
- 高可用、高并发、低延迟

{{< hint info >}}
考虑到数据量大到一定规模后都会进行归档。为了确保新生成的ID和归档中的ID也不一致，最简单的实现办法就是在ID中包含时间标识。
{{< /hint >}}

### 数据库自增主键

使用多台数据库分摊压力，每台起始值和步长=节点数。为了防止主从切换导致计数器丢失，需要将集群配置为半同步复制模式。

- 优点：实现简单；依靠数据库自身特性避免了页分裂。
- 缺点：依赖数据库性能；数据有序可被推测规模。

### 数据库+代理服务

在数据库中建立分布式主键表，字段包含max_id、step和biz_type。代理服务每次获取一批ID保存在内存中，同时修改数据库的max_id的值。应用通过代理服务的接口获取ID。代理服务可以通过双缓冲机制避免ID用完后加载新ID过程中引起的性能突刺。

- 优点：ID趋势递增。代理服务有缓存，DB宕机后也能工作一段时间。
- 缺点：ID有序。实现较复杂，引入了中间服务，存在性能损耗。

### 数据库+预置数据

每天日切后生成一批随机ID保存在数据库的未使用ID表中。应用或代理服务通过双缓冲机制获取ID，同时将表中ID移动到已使用ID表中。由于数据量够大，哪怕已使用ID表中的数据最终未被使用，也不用进行回滚。

- 优点：数据随机。长度短，如果选择Base64算法每一位表示64位数据，那么6位数字就足够表示数百亿数据。
- 缺点：实现复杂。数据无序，直接用作主键会引起页分裂。

### Snowflake算法

64位 = 保留位(1位) + 时间序列(41位) + Worker_ID(10位) + 自增序号(12位）

当同一机器两次获取的时间戳一致时，序列号就递增；当序列号用完后，程序就自旋直到下一个时间戳到来。为了防止时间漂移生成重复ID，需要引入ZooKeeper等手段保存上次ID的生成时间，当发生时钟回拨时不提供服务，进入自旋一直到服务器追上当前时间。

- 优点：高效，可以不依赖第三方组件（忽略时钟回拨的情况下）。
- 缺点：强依赖时钟，会有时钟回拨问题。默认实现下时间序列从1970年开始最多记录69年。数据长度太长。QPS不高情况下自增序号可能一直为1，无法作为分区键。
- 优化：
	+ 在QPS不高的情况下，可以将时间序列改为以秒为基准，确保自增序号不会总是1，或者每次时间拨动时自增序号的起始值做一下随机。
	+ 使用更紧凑的时间格式和机器ID。例如：20位 = yy（年的后2位） + mmddHHmmssSSS + IP第4位 + 4位序列号。

### MD5/Hash算法（幂等性）

为每个系统分配一个系统标识，一般3个英文字母即可。然后对（系统标识+数据）进行Hash计算，将结果作为分布式ID。

- 优点：实现简单。ID全局唯一。
- 缺点：数据无序，直接用作主键会引起页分裂。数据长度过长，MD5也有128位。同样的数据只会生成同样的ID。

在某些情况下可以用来实现幂等性，确保同样的数据不会被重复执行。

