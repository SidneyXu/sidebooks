---
weight: 1
title: 关系型数据库
---

# 数据库

## 知识点

建设中

## 最佳实践

### MySQL

- 什么情况下该创建索引
  - 列的区别度越高索引收益越大。
    + 查看区分度的方法： `SELECT COUNT(DISTINCT col_name)/COUNT(*) FROM table_name`。
  - 在单个索引就能实现查询效果的时候，也可以再建一个联合索引，通过实现覆盖索引和索引下推来提高查询性能。
- MySQL配置优化
  + 增加buffer_pool的大小到物理内存的60%-80%。
  - 增加buffer_pool实例的个数。
  - 根据读写性能调整io_threads的个数。
  - 增加change_buffer的大小。
  - 增加redo log buffer的大小，可能会增加丢数据的风险。
  - 磁盘TB级别，将redo log设为4个文件，每个1GB。
  - 增加binlog在page cache上的保存次数，可能会增加丢数据的风险。
  - 调整脏页的刷盘速度，使其符合磁盘IOPS。
    + IOPS测试方法：`fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest`
- CRUD性能优化
  * 架构级别
    - 数据库前放置缓存。
    - 分表分库。
    - 读写分离。
  * 表级别
    - 控制表的字段数量，默认情况下小于16全排序，大小16为rowId排序。
    - 控制字段大小。
    - 基于搜索条件和排序条件创建索引。
    - 只在索引上排序。
    - 普通索引优于唯一索引。
    - 时间和金钱可以考虑使用int和long。
  * SQL级别
    - 查询条件避免使用空值判断。
    - 小事务。
    - select + 具体字段。
    - 只要一行数据时使用LIMIT 1。MySQL数据库引擎会在查找到一条数据后停止搜索，而不是继续往后查询下一条符合条件的数据记录。
    - 事务内既有insert也有update时优先编写insert语句。行锁虽然只在必要时才会加上，但是在事务结束时才会被释放，将update语句放最后可以减少并发竞争。
    + `count(*)`。`count(*)`有优化，`count(1)`是每行记录返回1后统计1的数量。


MySQL Server 配置

```
# 连接数
## 服务器响应的最大连接数值（Max_used_connections）占服务器上限连接数值（max_connections）的比例值在10%以上，理想85%。5.7版本中默认是151， 最大可以达到16384。
## 如果只有一台服务器，且Tomcat设置的最大线程池缺省值200，假设每个线程会用到一个数据库连接，那么线程池大小应该小于等于200。
max_connections=5050

# redo log
## redo log每次都直接持久化到磁盘。
innodb_flush_log_at_trx_commit=1
## redo log buffer大小，默认为8MB。调高可减少磁盘写入次数。
innodb_log_buffer_size
## redo log日志大小
innodb_log_file_size=1G

# binlog
## binlog格式设为row。
binlog=row
## 1表示binlog每次都直接持久化到磁盘。提高性能时可设为100-1000。
## 该配置表示每次提交事务都write到文件系统的page cache，累计N个事务后才fsync到磁盘上。
## page cache是Linux的机制，MySQL重启这部分数据还在，但Linux重启数据就没了。
sync_binlog=1

# buffer pool、change_buffer
## buffer_pool大小，默认为128M。调大时可以显著提升系统性能，但是太大会引起页Swap。
## 通过SHOW GLOBAL STATUS LIKE 'innodb%read%';语句计算缓冲池的命中率
## 命中率=(1-innodb_buffer_pool_reads/innodb_buffer_pool_read_request)*100
## 如果池大小已经到达了物理内存的80%但命中率还很低，就需要调整物理内存大小。
innodb_buffer_pool_size=物理内存的60%-80%间。
## 50表示change_buffer占buffer pool的一半。
## 使用机械硬盘和存放历史数据的库使用change_buffer时收益很大，应该将其尽量增大。
innodb_change_buffer_max_size=50
## buffer_pool由多少个实例组成，可以减少竞争。只有innodb_buffer_pool_size>1G才生效，默认值为8。
## 大小不超过 innodb_read_io_threads + innodb_write_io_threads 之和
innodb_buffer_pool_instances
## buffer_pool实例的读写线程数，和与实例数相等性能最优
## SHOW GLOBAL STATUS LIKE 'Com_select';//读取数量
## SHOW GLOBAL STATUS WHERE Variable_name IN ('Com_insert', 'Com_update', 'Com_replace', 'Com_delete');//写入数量
## 读大于写，则读线程多一些，否则反之。
innodb_read_io_threads
innodb_write_io_threads

# 脏页
## 控制脏页的刷盘速度。应确保缓冲池中的脏页比例始终小于75%。
## 脏页太多会导致刷盘时间过长，影响查询时间。当数据从磁盘上载入到buffer_pool时，buffer_pool需要新开一个数据页，如果是干净页则直接释放该页数据继续使用。脏页证明内存和磁盘上数据不一致，需要先刷盘。
innodb_io_capacity=磁盘的IOPS
## SSD时设置，表示刷脏页时只刷当前页即可，为1表示当旁边也是脏页时则一起刷盘。
innodb_flush_neighbors=0

# 集群配置
## 并行复制时，在32核备库上设置为8到16，该值控制多少线程处理relaylog日志。提高该值可以缓解主备延迟，但需要留出CPU供备库的查询功能。
slave_parallel_workers=8或16
## 半同步复制
### 表示至少等待数据复制到几个从节点再返回。这个数量配置的越大，丢数据的风险越小，但是集群的性能和可用性就越差。最大可以配置成和从节点的数量一样，这样就变成了同步复制。一般配成默认值 1 也就够了。
rpl_semi_sync_master_wait_slave_count
### 主库提交事务的线程等待复制的时间超时了，这种情况下事务仍然会被正常提交。并且，MySQL 会自动降级为异步复制模式，直到有足够多（rpl_semi_sync_master_wait_no_slave）的从库追上主库，才能恢复成半同步复制。如果这个期间主库宕机，仍然存在丢数据的风险。
## 控制主库执行事务的线程，是在提交事务之前（AFTER_SYNC）等待复制，还是在提交事务之后（AFTER_COMMIT）等待复制。默认是 AFTER_SYNC，也就是先等待复制，再提交事务，这样完全不会丢数据。AFTER_COMMIT 具有更好的性能，不会长时间锁表，但是存在丢数据的风险。
rpl_semi_sync_master_wait_point
```


MySQL Client配置

```yml
spring:
  datasource:
    # 数据库连接池
    ## Spring默认的hikari速度最快
    ## 阿里的druid支持监控，功能更丰富，但监控可以由专门的监控系统负责
    hikari:
      # 最大连接数，默认为10，一般设置在20-30左右。
      maximum-pool-size: 20
```


### Oracle

Oracle Client 配置

```
loginTimeout=3000
checkoutTimeout=30000
preferredTestQuery=select 1 from dual
idleConnectionTestPeriod=10000
testConnectionOnCheckout=true
minPoolSize=5
maxPoolSize=15
initialPoolSize=1
acquireIncrement=1
acquireRetryAttempt=30
acquireRetryDelay=1000
maxIdleTime=25000
```


