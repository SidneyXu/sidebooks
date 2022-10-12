---
weight: 1
title: MySQL
---

# MySQL

## 集群方案

### MySQL Replication + MHA

美团MHA方案

MHA只负责主库的高可用。主库发生故障时，MHA会选择一个数据最接近原主库的候选主节点作为新的主节点，并补齐和之前宕机主库差异的Binlog。数据补齐之后，再将写VIP漂移到新主库上。

MHA脑裂问题

问题点：DB服务器的交换机出现了网络抖动，导致主库无法访问，被管理节点判定为故障，触发MHA切换，VIP被漂到了新主库上。随后交换机恢复，主库可被访问，但由于VIP并没有从主库上摘除，因此2台机器同时拥有VIP，同时提供服务，产生了脑裂现象。

解决方案：为MHA Manager加入了向同机架上其他物理机的进行探测的功能，让Manager通过对比更多的信息来判断是网络故障还是单机故障。

### MySQL Cluster

占位

### 基于Paxos的MGR版本的MySQL

占位


## 高级应用

### 查看当前所有连接

```sql
show processlist;
```

客户端如果太长时间没动静，连接器就会自动将它断开。这个时间由参数 wait_timeout 控制的，默认值是 8 小时。

```sql
 show variables like 'wait_timeout’;  
```

上面返回的单位为秒。如果在连接被断开之后，客户端再次发送请求的话，就会收到一个错误提醒： Lost connection to MySQL server during query。这时候如果就需要重连，再执行请求。

### Java层实现读写分离

1. 声明多个Datasource。
2. Dao层进行CRUD操作时通过ThreadLocal为当前线程注入枚举值。
3. 实现AbstractRoutingDataSource接口，在determineCurrentLookupKey()方法中根据ThreadLocal保存的枚举值返回对应的Datasource。


### 基数统计

通过基数统计可以判断当前索引添加是否合理。

```mysql
show index from <table_name>；
```

返回结果

![image](/images/db/mysql1.png)

- Cardinality：基数，表示字段的区分度。基数为采样统计，当表中变更的行数超过阀值时会触发重新采样。区分度越高索引越可能被MySQL优化器采用。

### 若MySQL误用索引

四种方式，推荐顺序由上到下。

- 通过explain命令进行分析，如果扫描行数rows和实际行数区别大，就执行`analyze table <table_name>` 对基数进行重新采样。
- 修改语句。
- 新建一个更合适的索引，让优化器做选择。或者允许的情况下删除误用的索引。
- 强制使用force_index。

### 为大表添加索引或字段

方法一、优先考虑类似 gh-ost 或者pt-online-schema-change这样的第三方方案，更加稳妥。

1. gh-ost创建与原始表相似的幽灵表“源表名_gho”。
2. 增量地将数据从原始表分批次插入复制到幽灵表中（默认1000条一次，--chunk-size=1000）。
3. 与此同时，另一个线程读取binlog将正在进行的更改操作同步到幽灵表中。
4. 最后锁定原始表，待binlog完全追上后，将原始表替换为ghost表。

方法二、MySQL 5.6 版本以后，使用Online DDL。

```mysql
ALTER TABLE t1 ADD COLUMN x INT, ALGORITHM=INPLACE;
```

方法三、在从库上执行操作，然后主从切换。传统方案，容易失误，但效率最高。

方法四、alter语句加时间。失败则间隔一段时间继续重试。

```mysql
alter table table_nam wait 10 add column`
```

方法五、建空表后通过触发器刷新数据。

```mysql
# 创建新表
CREATE TABLE main_table_new LIKE main_table;
# 在新表上添加字段或索引
ALTER TABLE main_table_new ADD COLUMN location VARCHAR(256);
# 复制旧表数据到新表
INSERT INTO main_table_new SELECT *, NULL FROM main_table;
# 重命名新表和旧表
RENAME TABLE main_table TO main_table_old, main_table_new TO main_table;
# 删除旧表
DROP TABLE main_table_old;
```

#### 健康检查

`select 1` 语句返回成功只能说明这个库的进程还在，并不能说明主库没问题。另外，当服务器日志满了后，还是能提供查询功能。所以查询表语句也不行。

解决方法为自建系统表

```mysql
update mysql.health_check set t_modified=now();
```

如果是双主架构，由于双主会互相将数据同步给对方，所以以上语句可能会发生冲突。可以在表上再添加server_id字段作为标识。


## SQL优化

### SQL分析

#### explain语句

通过 EXPLAIN 分析 SQL 执行计划，返回结果的字段含义如下所述：
   * id：每个执行计划都有一个 id。对于联合查询，这里还将有多个 id。
   * select_type：查询类型。包含SIMPLE（普通查询，即没有联合查询、子查询）、PRIMARY（主查询）、UNION（UNION 中后面的查询）、SUBQUERY（子查询）等。
   * table：查询的表。如果给表起别名了，则显示别名信息。
   * partitions：访问的分区表信息。
   * type：从表中查询到行所执行的方式。查询方式的结果值从好到差依次是：system > const > eq_ref > ref > range > index > ALL。
      - system/const：表中只有一行数据匹配，此时根据索引查询一次就能找到对应的数据。
      - eq_ref：使用唯一索引扫描，常见于多表连接中使用主键和唯一索引作为关联条件。
      - ref：非唯一索引扫描，还可见于唯一索引最左原则匹配扫描。
      - range：遍历一段范围的索引，常见于使用了 <，>，between 等操作。
      - index：遍历了整个索引树。
      - ALL：全表扫描。
   * possible_keys：可能使用到的索引。
   * key：实际使用到的索引。
   * key_len：当前使用的索引的长度。
   * ref：关联 id 等信息。
   * rows：扫描的行数。
   * filtered：结果集占总扫描记录数的比例。
   * Extra：额外信息。
   
explain 语句仅停留在分析阶段，不表示真实执行状态。


#### show profile语句(已弃用)

和explain纸上谈兵不同，show profile语句用于分析sql的具体执行性能。

```msql
# 开启性能分析
set profiling=0;
   
# 查询性能分析
show profiles；
   
# 查看具体的SQL执行步骤
show profile [type [, type] ... ]
   [FOR QUERY n]
   [LIMIT row_count [OFFSET offset]]
   ```

type参数：
   * ALL：显示所有开销信息
   * BLOCK IO：阻塞的输入输出次数
   * CONTEXT SWITCHES：上下文切换相关开销信息
   * CPU：显示CPU的相关开销信息
   * PC：接收和发送消息的相关开销信息
   * MEMORY ：显示内存相关的开销，目前无用
   * AGE FAULTS ：显示页面错误相关开销信息
   * SOURCE ：列出相应操作对应的函数名及其在源码中的调用位置(行数)
   * WWAPS：显示swap交换次数的相关开销信息
     
show profiles 默认情况下只显示最近执行的15 条SQL语句，可以通过设置`profiling_history_size`来增大该值，最大值为 100。不过该功能实际已被弃用，将来会被移除。官方推荐用performance_schema库的event_statment_**相关表替代，里面有87张表，部分功能默认开启。

#### performance_schema库

performance_schema库可用于分析SQL执行性能，找出长事务和慢查询。

查询长事务

可以在innodb_trx 这个表中查询长事务。比如下面这个语句，用于查找持续时间超过 60s 的事务。

```mysql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

#### 慢查询日志

通过以下命令行查询是否开启了记录慢 SQL 的功能，以及慢查询的阀值：

```msql
Show variables like 'slow_query%';
Show variables like 'long_query_time';
```

开启慢查询日志

```mysql
# 开启慢查询日志
set global slow_query_log='ON'; 
# 指定慢查询日志地址
set global slow_query_log_file='/var/lib/mysql/test-slow.log';
# 设置慢查询阀值
set global long_query_time=1;
```


#### pt-query-digest

建设中

### 处理慢SQL

定时检测方案

上线一个定时监控和杀掉慢 SQL 的脚本。这个脚本每分钟执行一次，检测有没有执行时间超过指定阀值（如一分钟）的慢 SQL，如果发现，直接杀掉这个会话。

降级方案

做一个简单的静态页面的首页作为降级方案，只要包含商品搜索栏、大的品类和其他顶级功能模块入口的链接就可以了。在 Nginx 上做一个策略，如果请求首页数据超时的时候，直接返回这个静态的首页作为替代。这样后续即使首页再出现任何的故障，也可以暂时降级，用静态首页替代。至少不会影响到用户使用其他功能。

### 历史数据清理

#### 删除数据后表空间没有释放的原因

和 InnoDB 的物理存储结构有关系。逻辑上虽然每个表是一颗 B+ 树，但是物理上每条记录都是存放在磁盘文件中，这些记录通过一些位置指针来组织成一颗 B+ 树。当 MySQL 删除一条记录的时候，只是把文件的这块区域标记为空闲，然后再修改 B+ 树中相关的一些指针，完成删除。实际被删除的记录还是存在。

#### 解决方案

方法一、建立新表

建立新表是最快的方式，但是需要停服。具体来说就是将数据复制到新表，然后改名。

方法二、在线迁移。

分批删除历史数据，可以基于主键删除进一步提高删除效率。

方法三、释放表空间

能接受锁表的话，可以执行一次 OPTIMIZE TABLE 释放存储空间。对于 InnoDB 来说，执行 OPTIMIZE TABLE 实际上就是把这个表重建一遍，执行过程中会一直锁表。重建的过程中，索引也会重建，所以表数据和索引数据都会更紧凑，不仅占用磁盘空间更小，查询效率也会有提升。

但该优化有个前提条件，MySQL 的配置必须是每个表独立一个表空间（innodb_file_per_table = ON）。


## 集群优化

### 分析主从延迟

在从库上执行`show slave status`可以看到从库落后主库多少秒（seconds_behind_master）。主备延迟的最直接表现是备库消费relay log的速度远低于主库生产bin log日志的速度。如果你需要实时监控主从延迟，可以用 pt-heartbeat 工具。 

从库延迟的原因

- 主从机器性能差异。
- 读写分离时没有关注读库的性能。
   + 可以多接几个读库或者让读库多接几个从库分摊压力。
- 大事务。主库事务执行多长，从库之后也要执行多长时间。
   + 不要一次性删除大量数据，特别是历史数据，要分多个事务分批批删。
- 在大表上执行DDL。
   + 使用gh-ost等方案。

### 手动主备切换

1. 备库 readonly=true，只读。
2. 在备库查看 seconds_behind_master，低于5秒则执行下一步，否则重试。
3. 设置主库 readonly=true，只读。
4. 在备库查看 seconds_behind_master，等于0则执行下一步，否则重试。
5. 把备库 readonly=false。
6. 把业务切换到备库。

### 手动一主多从切换

例如主机A和备机A`互备，从机B、C、D作为主机A的从机，提供只读功能。

使用GTID进行切换

MySQL 5.6 版本引入的 GTID 模式。GTID 的全称是 Global Transaction Identifier，也就是全局事务 ID，是一个事务在提交的时候生成的，是这个事务的唯一标识。

格式是：GTID=server_uuid:gno

server_uuid 是一个实例第一次启动时自动生成的，是一个全局唯一的值；gno 是一个整数，初始值是 1，每次提交事务的时候分配给这个事务，并加 1。

GTID 模式的启动也很简单，我们只需要在启动一个 MySQL 实例的时候，加上参数 gtid_mode=on 和 enforce_gtid_consistency=on 就可以了。





