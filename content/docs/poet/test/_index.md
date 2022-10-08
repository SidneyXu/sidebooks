---
weight: 120
title: 测试与性能分析
bookCollapseSection: true
---

# 性能分析大纲

## 什么时候开始调优

- 项目开发初期更注重开发进度，完成业务功能为先。
- 开发完成后，根据产品预估的线上数据规模进行性能测试，查看性能是否在预期范围内。
- 项目上线后，根据运维平台的日志监控以及性能统计日志检测分析系统性能。

## 压力测试工具

- Apache的ab。注意Mac自带的有并发数量限制，需要重新编译。
- 基于Java的JMeter
- 基于Scala的Gatling
- 手写Python

## 性能测试指标

- CPU性能
  - 利用率：单位时间内一个线程或进程实时占用CPU的百分比。
  - 系统负载：单位时间内正在运行或等待的进程或线程数。代表系统繁忙程度。
  - 利用率和负载的关系
    - CPU密集型系统负载未必会高，但CPU利用率肯定高。
    - I/O密集型CPU利用率可能不高，但系统负载会很高，因为有很多线程会等待I/O读写完毕。
    - CPU利用率高负载低：可能计算复杂但任务就一个。
    - CPU利用率低负载高：可能计算简单但等待I/O的任务数量多。
- 可用性
  - 99.99%：4个9表示全年不可用时间少于52.6分钟。用于核心服务指标。
  - 99.9%：3个9表示全年不可用时间少于年8小时。用于一般服务指标。
  - RTO：RecoveryTimeObjective，故障恢复时间。
  - RPO：RecoveryPointObjective，故障恢复程度。如果数据时按日备份则最多丢失1天数据。
- 接口性能
  - QPS：接口的一次请求到服务器的返回结果。
    - PV=预估用户数UV  x 每用户访问接口数量，PV/8小时=QPS。
  - TPS：代表一次用户操作到服务器的返回结果，可以包含多个请求。
  - RT：响应时间。常见指标为AVG平均响应、95线和99线。
  - 99%：99%个请求所表示的性能范围。95线和99线与AVG线越相近证明性能越平稳。99线可能是由于该用户使用系统最频繁的用户，因此数据最多，性能最慢，所以也不能忽略其响应速度。
- 数据库性能
  - 执行时间。
- GC性能
  - 停顿时间（延迟）：停顿时间越短响应速度越高，用户体验越好。由于多线程GC的原因，一般real远低于user+sys。
    - 调小GC开始阀值。
    - 增加并发标记线程数量。
    - 调整停顿时间。
    - 控制堆大小。
    - 增加机器内存大小。
    - 增加年轻代大小。
    - 降低老年代提升次数。
    - 加大脏卡大小。
    - 减少I/O数量。
  - 吞吐量：吞吐量越高越能更快完成任务，特别是后台长时间批处理任务。通常应该达到95%以上。
    - 增加GC开始阀值。
    - 减少并发标记线程数量。
    - 调整吞吐量大小。
  - 对象创建速率：速度越快越可能发生GC。
  - STW：CMS在YGC、初始标记、最终标记、FGC时会产生STW。G1在YGC、初始标记、标记、Mixed、清理、FGC时会产生STW。

## 性能分析工具

### 综合性能分析工具

Arthas

- thread命令可排查线程死锁、CPU过高等问题。
- sc可以查找加载的类信息，sm可以查找加载的函数信息，可以用于排查线上jar包版本是否正确。
- jad可反编译代码。
- watch可监测指定方法的参数、返回值和异常信息。
- trace可监测方法的调用链，展示每个调用的耗时，找出最耗时的方法。
- profiler可生成火焰图，监测CPU运行情况和函数调用频率。
- ognl可配合用来执行动态代码。

### GC性能分析工具

- jstat
  - `jstat -gcutil <pid> <sec>` 查看应用启动到现在的GC情况
- jmap
  - `jmap -heap <pid>`
- GCeasy
- GCViewer

### CPU性能分析工具

- async-profiler 火焰图。通过采样可以分析出调用次数最多的函数。

### 内存性能分析工具

- jmap：通过比较多次dump的内存数据可以找出内存泄露代码。
- VisualVM
- MAT

### 线程性能分析工具

- jstack：查看线程堆栈和死锁信息。

## 现象分析

### GC性能

#### real > user + sys

原因分析：

- 后台磁盘I/O繁忙，引起GC日志暂停。
  - 减少机器上其它进程数量。减少日志数量。将日志记录在单独的HDD或SSD磁盘上。
- 没有足够的CPU资源来运行。
  - 减少机器上其它进程数量。

#### sys > user

原因分析：

- 操作系统出现问题。
  - 检查内存、磁盘、操作系统是否工作正常。
- 虚拟机没有足够资源运行。
- 内存不足，没有连续的页空间，引起页移动。
- 磁盘I/O压力过大。

#### 垃圾回收原因

- Allocation Failure：年轻代没有足够空间分配内存。
  - 增加年轻代大小。
- Promotion Failure：年轻代对象无法提示到老年代，可能老年代空间不足或存在大量碎片，必须STW。
- Concurrent Mode Failure：并发回收失败。CMS无法及时完成对象的收集就会退化为旧的并行收集器进行FGC。
  - 增加老年代大小。
  - 设置 `CMSInitiatingOccupancyFraction` 为较低的值让CMS提前启动以及将`-XX:+UseCMSInitiatingOccupancyOnly` 设为true让CMS每次都采用设定的阀值。需要注意如果`CMSInitiatingOccupancyFraction` 太低就会频繁进行GC。
- Metadata GC Threshold：元数据GC阀值。元空间太小或存在类加载器泄露（可能性很小）。
  - 设置 `-XX:MetaspaceSize` 来增加元空间大小。
- G1 Humongous Allocation：G1大对象分配。大对象指大小超过region一半的对象。频繁分配大对象会导致两个性能问题：大对象不足region空间的部分成为碎片，不会被使用；JDK 1.8u40之前大对象空间只在FGC时回收
  - 设置 `-XX:G1HeapRegionSize` 增加region大小，避免对象超过region的50%成为大对象，值需要为2的倍数，且范围为1-32mb之间。默认情况下，region大小是在启动期间根据堆大小计算的。
- G1 Evacuation Pause：G1疏散暂停，发生在region复制时。YGC和Mixed GC时会将存活对象从一组region复制到另一组region，如果此时空间不足就会触发FGC。
  - 内存方面只设置堆的最小或最大空间以及暂停时间，避免手动设置年轻代和老年代大小。
  - 如果问题仍然存在，增加堆大小。
  - 如果无法增加堆大小，并且标记周期开始过晚，可以减少 `-XX:InitiatingHeapOccupancyPercent` 的值。默认值为45%，表示内存利用达到45%时才开始标记循环。如果标记周期开始较早且未回收，则将 `-XX:InitiatingHeapOccupancyPercent` 阈值提高到默认值以上。
  - 设置 `-XX:ConcGCThreads` 来增加并发标记的线程数量。
  - 设置 `-XX:G1ReservePercent` 参数的值。默认值为10%，上限为50%。这意味着G1垃圾收集器将始终保持10%的内存可用。增加此值时，GC将提前触发，从而阻止疏散暂停。

#### 调优参数

调优策略

- 降低Minor GC频率：增大新生代空间，标记时间的成本低于复制的成本
- 降低FGC频率：增加堆内存大小，减少大对象创建
- Biz日志影响GC

通用参数

- 最小堆最大堆设为一样，4G到8G间；
- 预分配内存；
- 降低老年代提升次数为3；
- 加大脏卡大小为1024;
- GC日志写入单独磁盘上；
- 不独占机器时，降低并行线程数量；
- 并发标记线程数增加为并行线程数的一半；
- 增大 Integer Cache；

CMS参数

- 新生代老年代配为1比1；
- 内存满75%就开始CMS；
- System.gc()也使用CMS；

G1参数

- 延迟时间再200ms（默认）到500ms之间；
- 调整吞吐量，默认为12，92%时间在应用上。20表示95%时间应用于应用线程；
- 删除存活时间较长的重复字符串；
- 调整Mixed GC标记起始阀值，默认为45%；

### 接口性能

#### 应用长暂停

请求未在合理时间内返回。

原因分析：

- 对象创建速率过高，引起频繁GC。
  - 控制对象创建速率，复用对象。
- 年轻代太小，过早提升到老年代。
  - 增加年轻代大小。
- 内存不足，发生磁盘I/O的进程切换。
  - 减少机器上其它进程数量，或增加内存大小。
- real近似user+sys，GC线程太少。
  - 增加GC线程数量。
- 后台磁盘I/O繁忙，引起GC日志暂停。
  - 减少机器上其它进程数量。
- 显式`System.gc()`采用了FGC。
  - 可配置 ``-XX:+DisableExplicitGC` ` 禁用显式GC。
- 堆空间过大，每次扫描时间太长。
  - 控制堆大小最好为8G-16G。
- GC工作量分配不均匀，如数据结构不支持并发扫描，并发分配失败等。
  - 可配置 `-XX:+CMSScavengeBeforeRemark`

### 内存溢出

#### 内存泄露

指对象无法被正确回收，随着时间流逝越来越多。

检查方法：

- jmap多次dump后，对比数据。泄露的对象会越来越多，不会被回收。