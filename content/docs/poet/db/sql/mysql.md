---
weight: 1
title: MySQL
---

## MySQL

### 1、基本使用

#### 1.1、查看当前所有连接

```mysql
show processlist;
```

客户端如果太长时间没动静，连接器就会自动将它断开。这个时间是由参数 wait_timeout 控制的，默认值是 8 小时。

```mysql
 show variables like 'wait_timeout’;  
```

上面返回的单位为秒。如果在连接被断开之后，客户端再次发送请求的话，就会收到一个错误提醒： Lost connection to MySQL server during query。这时候如果你要继续，就需要重连，然后再执行请求了。

### 2、高可用方案

##### 2.1、MySQL Replication + MHA

美团MHA
MHA只负责MySQL主库的高可用。主库发生故障时，MHA会选择一个数据最接近原主库的候选主节点（这里只有一个从节点，所以该从节点即为候选主节点）作为新的主节点，并补齐和之前Dead Master 差异的Binlog。数据补齐之后，即将写VIP漂移到新主库上。

MHA脑裂问题
DB服务器的上联交换机出现了抖动，导致主库无法访问，被管理节点判定为故障，触发MHA切换，VIP被漂到了新主库上。随后交换机恢复，主库可被访问，但由于VIP并没有从主库上摘除，因此2台机器同时拥有VIP，会产生脑裂。我们对MHA Manager加入了向同机架上其他物理机的探测，通过对比更多的信息来判断是网络故障还是单机故障。

##### 2.2、MySQL Cluster

##### 2.3、基于Paxos的MGR版本的MySQL

### 3、性能优化

#### 3.1、索引设计

该在哪种字段上添加索引

```mysql
show index from <table_name>；
```

![image](/images/db/mysql1.png)

- Cardinality：基数，表示字段的区分度。基数为采样统计，当变更的行数超过阀值时会触发重新采样。区分度越高索引越可能被使用。

当MySQL误用索引时

- 如果explain分析，如果扫描行数rows和实际行数区别大，就执行`analyze table <table_name>` 对基数重新采样。
- 考虑修改语句。
- 新建一个更合适的索引，让优化器做选择。或者在某种场景下删除误用的索引。
- 强制使用force_index。

列的区别度越高索引收益越大。查看区分度的方法为 `SELECT COUNT(DISTINCT col_name)/COUNT(*) FROM table_name`。

#### 3.2、为大表添加索引或字段

1、优先考虑类似 gh-ost 或者pt-online-schema-change这样的第三方方案，更加稳妥。

原理：

Gh-ost创建与原始表相似的幽灵表“源表名_gho”，增量地将数据从原始表分批次插入复制到幽灵表中（默认1000条一次，--chunk-size=1000），同时另一个线程读取binlog将正在进行的更改传播到幽灵表中。最后锁定原表，待binlog完全追上，将原始表替换为ghost表。

2、MySQL 5.6 版本以后，创建索引都支持 Online DDL 了。可以用于小表加索引。

ALTER TABLE t1 ADD COLUMN x INT, ALGORITHM=INPLACE;

INPLACE 算法，从 MySQL 5.6 开始被引入并默认使用。

3、传统方案，在从库上执行操作，然后主从切换。

一主一备，主库 A、备库 B，步骤如下：

1. 在备库 B 上执行 set sql_log_bin=off，也就是不写 binlog，然后执行 alter table 语句加上索引；
2. 执行主备切换；这时候主库是 B，备库是 A。
3. 在 A 上执行 set sql_log_bin=off，然后执行 alter table 语句加上索引。

在需要紧急处理时，这个方案的效率是最高的。

4、Alter 加时间，防止MDL影响线上数据读取。`alter table table_nam wait 10 add column`。

大表加字段：建新表转移数据后重命名表名；alter+时间；online ddl；

5、建空表通过触发器刷新数据

```mysql
CREATE TABLE main_table_new LIKE main_table;

ALTER TABLE main_table_new ADD COLUMN location VARCHAR(256);

INSERT INTO main_table_new SELECT *, NULL FROM main_table;

RENAME TABLE main_table TO main_table_old, main_table_new TO main_table;

DROP TABLE main_table_old;
```

#### 3.3、SQL优化

优化 SQL 语句的步骤

1. 通过 EXPLAIN 分析 SQL 执行计划

   * id：每个执行计划都有一个 id，如果是一个联合查询，这里还将有多个 id。

   * select_type：表示 SELECT 查询类型，常见的有 SIMPLE（普通查询，即没有联合查询、子查询）、PRIMARY（主查询）、UNION（UNION 中后面的查询）、SUBQUERY（子查询）等。

   * table：当前执行计划查询的表，如果给表起别名了，则显示别名信息。

   * partitions：访问的分区表信息。

   * type：表示从表中查询到行所执行的方式，查询方式是 SQL 优化中一个很重要的指标，结果值从好到差依次是：system > const > eq_ref > ref > range > index > ALL。

   * system/const：表中只有一行数据匹配，此时根据索引查询一次就能找到对应的数据。

   * eq_ref：使用唯一索引扫描，常见于多表连接中使用主键和唯一索引作为关联条件。

   * ref：非唯一索引扫描，还可见于唯一索引最左原则匹配扫描。

   * range：索引范围扫描，比如，<，>，between 等操作。

   * index：索引全表扫描，此时遍历整个索引树。

   * ALL：表示全表扫描，需要遍历全表来找到对应的行。

   * possible_keys：可能使用到的索引。

   * key：实际使用到的索引。

   * key_len：当前使用的索引的长度。

   * ref：关联 id 等信息。

   * rows：查找到记录所扫描的行数。

   * filtered：查找到所需记录占总扫描记录数的比例。

   * Extra：额外的信息。


2. 通过 Show Profile 分析 SQL 执行性能。通过 EXPLAIN 分析执行计划，仅仅是停留在分析 SQL 的外部的执行情况，如果我们想要深入到 MySQL 内核中，从执行线程的状态和时间来分析的话，这个时候我们就可以选择 Profile。

   ```msql
   set profiling=0;
   
   show profiles；
   
   # 查看具体的步骤时间
   SHOW PROFILE [type [, type] ... ]
   [FOR QUERY n]
   [LIMIT row_count [OFFSET offset]]
   ```


   type参数：

   * | ALL：显示所有开销信息

   * | BLOCK IO：阻塞的输入输出次数

   * | CONTEXT SWITCHES：上下文切换相关开销信息

   * | CPU：显示CPU的相关开销信息

   * | IPC：接收和发送消息的相关开销信息

   * | MEMORY ：显示内存相关的开销，目前无用

   * | PAGE FAULTS ：显示页面错误相关开销信息

   * | SOURCE ：列出相应操作对应的函数名及其在源码中的调用位置(行数)

   * | SWAPS：显示swap交换次数的相关开销信息
     show Profiles 只显示最近发给服务器的 SQL 语句，默认情况下是记录最近已执行的 15 条记录，我们可以重新设置 profiling_history_size 增大该存储记录，最大值为 100。另外该功能实际已被弃用，将来会被移除。官方推荐用performance_schema库的event_statment_**相关表替代，里面有87张表，部分功能默认开启。

#### 3.4、慢查询分析

在开发阶段，衡量一个 SQL 查询语句查询性能的手段是，估计执行 SQL 时需要遍历的数据行数。遍历行数在百万以内，可以认为是安全的 SQL，百万到千万这个量级则需要仔细评估和优化，千万级别以上则是非常危险的。为了减少慢 SQL 的可能性，每个数据表的行数最好控制在千万以内。

1、慢查询日志

通过以下命令行查询是否开启了记录慢 SQL 的功能，以及最大的执行时间是多少：

```msql
Show variables like 'slow_query%';
Show variables like 'long_query_time';
```

开启慢查询日志

```mysql
set global slow_query_log='ON'; //开启慢SQL日志
set global slow_query_log_file='/var/lib/mysql/test-slow.log';//记录日志地址
set global long_query_time=1;//最大执行时间
```

2.、information_schema 

查询长事务

可以在 information_schema 库的 innodb_trx 这个表中查询长事务，比如下面这个语句，用于查找持续时间超过 60s 的事务。

```mysql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

3、pt-query-digest

#### 3.5、处理慢SQL

定时检测

上线一个定时监控和杀掉慢 SQL 的脚本。这个脚本每分钟执行一次，检测有没有执行时间超过一分钟的慢 SQL，如果发现，直接杀掉这个会话。

降级

做一个简单的静态页面的首页作为降级方案，只要包含商品搜索栏、大的品类和其他顶级功能模块入口的链接就可以了。在 Nginx 上做一个策略，如果请求首页数据超时的时候，直接返回这个静态的首页作为替代。这样后续即使首页再出现任何的故障，也可以暂时降级，用静态首页替代。至少不会影响到用户使用其他功能。

#### 3.6、历史数据清理

1、表空间没有释放的原因

和 InnoDB 的物理存储结构有关系。虽然逻辑上每个表是一颗 B+ 树，但是物理上，每条记录都是存放在磁盘文件中的，这些记录通过一些位置指针来组织成一颗 B+ 树。当 MySQL 删除一条记录的时候，只是把文件的这块区域标记为空闲，然后再修改 B+ 树中相关的一些指针，完成删除。其实那条被删除的记录还是躺在那个文件的那个位置，所以并不会释放磁盘空间。

2、建立新表

建立新表是最快的方式，但是需要停服。具体来说就是将数据复制到新表，然后改名。

```mysql
-- 新建一个临时订单表
create table orders_temp like orders;
-- 把当前订单复制到临时订单表中
insert into orders_temp
select * from orders
where timestamp >= SUBDATE(CURDATE(),INTERVAL 3 month);
-- 修改替换表名
rename table orders to orders_to_be_droppd, orders_temp to orders;
-- 删除旧表
drop table orders_to_be_dropp
```

3、在线迁移。

分批删除历史订单数据，避免给线上数据库造成太大压力，可以基于主键删除进一步提高删除效率。

4、释放表空间

如果磁盘空间很紧张，非要把这部分磁盘空间释放出来，可以执行一次 OPTIMIZE TABLE 释放存储空间。

对于 InnoDB 来说，执行 OPTIMIZE TABLE 实际上就是把这个表重建一遍，执行过程中会一直锁表。另外，这么优化有个前提条件，MySQL 的配置必须是每个表独立一个表空间（innodb_file_per_table = ON），如果所有表都是放在一起的，执行 OPTIMIZE TABLE 也不会释放磁盘空间。

重建表的过程中，索引也会重建，这样表数据和索引数据都会更紧凑，不仅占用磁盘空间更小，查询效率也会有提升。那对于频繁插入删除大量数据的这种表，如果能接受锁表，定期执行 OPTIMIZE TABLE 是非常有必要的。

### 4、集群优化

#### 4.1、主备延迟

在备库上执行`show slave status`可以看到备库落后主库多少秒（seconds_behind_master）。主备延迟的最直接表现是备库消费relay log的速度远低于主库生产bin log日志的速度。

如果只是看一下，可以连接到主库上用 show slave status 命令查看
如果你需要实时监控主从延迟，可以用 pt-heartbeat 工具。 

备库延迟的原因

- 主备机器性能差异。
- 读写分离，没有关注读库的性能。可以多接几个读库或者让读库多接几个从库分摊压力。
- 大事务。主库执行10min事务写入binlog后，备库也要执行同样时间。所以不要一次性删除大量数据，特别是历史数据，要分多个事务分批批删。
- 大表DDL。应使用gh-ost方案。

#### 4.2、主备切换

手动主备切换流程

1. 备库 readonly=true，只读。
2. 在备库查看 seconds_behind_master，低于5秒则执行下一步，否则重试。
3. 设置主库 readonly=true，只读。
4. 在备库查看 seconds_behind_master，等于0则执行下一步，否则重试。
5. 把备库 readonly=false。
6. 把业务切换到备库。

#### 4.3、一主多从切换

例如主机A和备机A`互备，从机B、C、D作为主机A的从机，提供只读功能。

使用GTID进行切换

GTID 的全称是 Global Transaction Identifier，也就是全局事务 ID，是一个事务在提交的时候生成的，是这个事务的唯一标识。

格式是：GTID=server_uuid:gno

server_uuid 是一个实例第一次启动时自动生成的，是一个全局唯一的值；gno 是一个整数，初始值是 1，每次提交事务的时候分配给这个事务，并加 1。

GTID 模式的启动也很简单，我们只需要在启动一个 MySQL 实例的时候，加上参数 gtid_mode=on 和 enforce_gtid_consistency=on 就可以了。

MySQL 5.6 版本引入的 GTID 模式。

#### 4.4、主库的健康检查

`select 1` 语句返回成功只能说明这个库的进程还在，并不能说明主库没问题。另外，当服务器日志满了后，还是能提供查询功能。所以查询表语句也不行。

解决方法为自建系统表

```mysql
update mysql.health_check set t_modified=now();
```

如果是双主，由于双主会互相将数据同步给对方，所以以上语句可能会发生冲突。可以在表上再添加server_id字段作为标识。

### 5、使用方法

#### 5.1、Java层读写分离

- 定义多个Datasource
- save等操作时通过ThreadLocal注入枚举
- 实现AbstractRoutingDataSource接口，在方法determineCurrentLookupKey根据枚举返回对应的Datasource