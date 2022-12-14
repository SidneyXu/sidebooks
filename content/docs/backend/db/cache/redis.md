---
title: Redis
weight: 1
---

# Redis

## 性能优化

### 如何计算Redis所占内存空间

在网站 http://www.redis.cn/redis_memory/
可以在线计算Redis占用的内存空间。

### 如何判断Redis性能是否出了问题

使用基线性能判断。基线性能是系统在低压力、无干扰下的基本性能，这个性能只由软硬件配置决定。

```shell
./redis-cli --intrinsic-latency 120
```

输出结果

![image](/images/db/redis1.png)


以上命令会打印 120 秒内监测到的最大延迟。一般情况下，运行 120 秒就足够监测到最大延迟了。如果运行时延迟是基线性能的 2 倍以上，就可以认定 Redis 变慢了。

### 如何排查 Redis 的 bigkey

redis 可以在执行 redis-cli 命令时带上`–bigkeys `选项，进而对整个数据库中的键值对大小情况进行统计分析，比如说，统计每种数据类型的键值对个数以及平均大小。

```shell
./redis-cli  --bigkeys

-------- summary -------
Sampled 32 keys in the keyspace!
Total key length in bytes is 184 (avg len 5.75)

//统计每种数据类型中元素个数最多的bigkey
Biggest   list found 'product1' has 8 items
Biggest   hash found 'dtemp' has 5 fields
Biggest string found 'page2' has 28 bytes
Biggest stream found 'mqstream' has 4 entries
Biggest    set found 'userid' has 5 members
Biggest   zset found 'device:temperature' has 6 members

//统计每种数据类型的总键值个数，占所有键值个数的比例，以及平均大小
4 lists with 15 items (12.50% of keys, avg size 3.75)
5 hashs with 14 fields (15.62% of keys, avg size 2.80)
10 strings with 68 bytes (31.25% of keys, avg size 6.80)
1 streams with 4 entries (03.12% of keys, avg size 4.00)
7 sets with 19 members (21.88% of keys, avg size 2.71)
5 zsets with 17 members (15.62% of keys, avg size 3.40)
```

该工具是通过扫描数据库来查找 bigkey 的。所以在执行的过程中，会对 Redis 实例的性能产生影响，建议在从节点上执行。注意对于集合类型来说，这个方法只统计集合元素个数的多少，而不是实际占用的内存量。

### 碎片整理

#### 碎片原因

Redis 释放的内存空间会由内存分配器管理，并不会立即返回给操作系统，所以会产生内存碎片。内存碎片会导致Redis虽然有空闲空间，但无法用来保存数据。

内存碎片的形成有内因和外因两个层面的原因。内因是操作系统的内存分配机制，外因是 Redis 的负载特征。

- 内因：内存分配器的分配策略决定了操作系统无法做到“按需分配”。因为内存分配器一般是按固定大小来分配内存，而不是完全按照应用程序申请的内存空间大小给程序分配。这样的分配方式本身是为了减少分配次数。
- 外因：不同类型键值对大小不一样，频繁删改操作会引起扩容和释放。

#### 检查内存碎片

Redis 自身提供了 INFO 命令来查看内存使用情况。

```shell
INFO memory
# Memory
used_memory:1073741736
used_memory_human:1024.00M
used_memory_rss:1997159792
used_memory_rss_human:1.86G
…
mem_fragmentation_ratio:1.86
```

- mem_fragmentation_ratio ：内存碎片率，约等于 used_memory_rss/used_memory ，mem_fragmentation_ratio 大于 1.5 。这表明内存碎片率已经超过了 50%。
   + 如果mem_fragmentation_ratio小于1，说明used_memory_rss小于了used_memory。意味着操作系统分配给Redis进程的物理内存，要小于实际存储数据的内存，这样会因内存不足导致Swap中。
- used_memory_rss：操作系统实际分配给 Redis 的物理内存空间，里面包含了碎片。
- used_memory：Redis 实际申请使用的空间。


#### 处理内存碎片

方法一、重启 Redis 实例。

方法二、从 4.0-RC3 版本以后，开启内存碎片自动清理。

Redis 需要启用自动内存碎片清理。

```shell
config set activedefrag yes
```

如果同时满足下面两个条件，就开始清理。清理的过程中，只要有一个条件不满足了，就停止自动清理。

- active-defrag-ignore-bytes 100mb：内存碎片的字节数达到 100MB 时开始清理；
- active-defrag-threshold-lower 10：内存碎片空间占操作系统分配给 Redis 的总空间比例达到 10% 时开始清理。

为了尽可能减少碎片清理对 Redis 的影响，自动内存碎片清理功能在执行时，还会监控清理操作占用的 CPU 时间，控制清理操作占用的 CPU 时间比例的上、下限，既保证清理工作能正常进行，又避免了降低 Redis 性能。这两个参数具体如下：

- active-defrag-cycle-min 25： 自动清理过程所用 CPU 时间的比例不低于 25%，保证清理能正常开展；
- active-defrag-cycle-max 75：自动清理过程所用 CPU 时间的比例不高于 75%，一旦超过，就停止清理，从而避免在清理时，大量的内存拷贝阻塞 Redis，导致响应延迟升高。

内存碎片自动清理涉及内存拷贝，发生在主线程，是一个性能风险点。如果 Redis 性能变慢，且日志表示正在进行碎片清理。可以调小 active-defrag-cycle-max 的值，以减轻对正常请求处理的影响。


### 慢查询日志

查看慢查询日志，n为显示条数。

```shell
SLOWLOG GET <n>
1) 1) (integer) 33           //每条日志的唯一ID编号
   2) (integer) 1600990583   //命令执行时的时间戳
   3) (integer) 20906        //命令执行的时长，单位是微秒。这里大约20毫米，算比较慢。
   4) 1) "keys"               //具体的执行命令和参数。这里表示命令keys abc*。
      2) "abc*"
   5) "127.0.0.1:54793"      //客户端的IP和端口号
   6) ""                     //客户端的名称，此处为空
```

- slowlog-log-slower-than：控制慢查询日志记录大于多少微秒的命令。
- slowlog-max-len：最多记录多少条命令记录。底层实现是一个具有预定大小的先进先出队列，一旦记录的命令数量超过了队列长度，最先记录的命令操作就会被删除。默认为 128。一般建议设置为 1000 左右。


### latency monitor

设置阀值，超过该阀值的命令会被记录下来。

```shell
config set latency-monitor-threshold 1000
```

查看最新和最大的超过阈值的延迟情况

```shell
latency latest
1) 1) "command"
   2) (integer) 1600991500    //命令执行的时间戳
   3) (integer) 2500           //最近的延迟执行时间
   4) (integer) 10100          //最大的延迟持续时间
```

## 集群架构

### 高可用架构

APP -> Redis Sentinel(3个) -> Redis Server Master/Slave，即哨兵3台，Server2台，一主一从

### 脑裂问题

主要问题在于防止旧主库在主从切换过程中接收客户端请求。解决方案为当以下配置不符合时，主库就不会提供服务。

- min-slaves-to-write：主库能进行数据同步的最少从库数量；
- min-slaves-max-lag：主从库间进行数据复制时，从库给主库发送 ACK 消息的最大延迟时间（以秒为单位）。

假设将 min-slaves-to-write 设置为 1，把 min-slaves-max-lag 设置为 12s，把哨兵的 down-after-milliseconds 设置为 10s，主库因为某些原因卡住了 15s，导致哨兵判断主库客观下线，开始进行主从切换。同时，因为原主库卡住了 15s，没有一个从库能和原主库在 12s 内进行数据复制，原主库也无法接收客户端请求了。这样一来，主从切换完成后，也只有新主库能接收请求，不会发生脑裂，也就不会发生数据丢失的问题了。

假设从库有 K 个，可以将 min-slaves-to-write 设置为 K/2+1（如果 K 等于 1，就设为 1），将 min-slaves-max-lag 设置为十几秒（例如 10～20s），在这个配置下，如果有一半以上的从库和主库进行的 ACK 消息延迟超过十几秒，就禁止主库接收客户端写请求。

### 分片集群

- cluster-node-timeout ：设置了 Redis Cluster 中实例响应心跳消息的超时时间。Redis Cluster 正常运行的条件是，有半数以上的实例都能正常运行。如果执行主从切换的实例超过半数，而主从切换时间又过长的话，就可能有半数以上的实例心跳超时，从而可能导致整个集群挂掉。所以，应将 cluster-node-timeout 调大些（例如 10 到 20 秒）。

### 复制积压缓冲区的溢出问题

主节点在和从节点同步时会先把命令写入复制积压缓冲区。一旦从节点发生网络闪断，再次和主节点恢复连接后，从节点就会从复制积压缓冲区中读取断连期间主节点接收到的写命令，进行增量同步。

复制积压缓冲区是一个大小有限的环形缓冲区。当主节点把复制积压缓冲区写满后，会覆盖缓冲区中的旧命令数据。
如果从节点还没有同步这些旧命令数据，就会造成主从节点间重新开始执行全量复制。

为了应对复制积压缓冲区的溢出问题，我们可以调整复制积压缓冲区的大小，也就是设置 repl_backlog_size 这个参数的值。

### 复制缓冲区的溢出问题

在全量复制过程中，主节点在向从节点传输 RDB 文件的同时，会继续接收客户端发送的写命令请求。这些写命令会先保存在复制缓冲区中，等 RDB 文件传输完成后，再发送给从节点去执行。主节点上会为每个从节点都维护一个复制缓冲区，来保证主从节点间的数据同步。

所以，如果在全量复制时，从节点接收和加载 RDB 较慢，同时主节点接收到了大量的写命令，写命令在复制缓冲区中就会越积越多，最终导致溢出。
