---
weight: 120
title: 测试与性能分析
bookCollapseSection: true
---

# 测试与性能分析

## 性能分析

### 什么时候开始性能分析和调优

- 项目开发初期应注重开发进度，以完成业务功能为先。
- 项目开发完成后，可以根据运营和产品预估的线上数据规模进行性能测试，检查当前系统性能是否满足要求。
- 项目上线后，根据监控数据进行扩容、缩容、调整参数等后续处理。

### 性能分析指标

CPU性能
* CPU利用率：单位时间内线程或进程实时占用CPU的百分比。
* CPU负载：单位时间内正在运行或等待的进程或线程数，代表系统的繁忙程度。
* 利用率和负载的关系
  * CPU密集型系统负载未必会高，但CPU利用率肯定高。
  * I/O密集型CPU利用率可能不高，但系统负载会很高。
  * 如果CPU利用率高但系统负载低，可能待执行任务数量少，但计算量大且复杂。
  * 如果CPU利用率低但系统负载高，可能计算简单，但等待I/O的任务数量众多。

可用性
* 99.99%：4个9表示全年不可用时间少于52.6分钟。一般作为核心服务的指标。
* 99.9%：3个9表示全年不可用时间少于年8小时。一般作为普通服务的指标。
* RTO（RecoveryTimeObjective）：故障恢复时间。
* RPO（RecoveryPointObjective）：故障恢复程度。如果数据按日备份则最多丢失1天数据。

接口性能
* QPS（Query Per Second）：一次接口请求到服务器的返回结果。
  * 计算公式：`PV = UV x 每用户访问接口数量；QPS = PV / 运行时间`；运行时间可取8小时作为平日QPS，取3小时作为峰值QPS。
* TPS（Transaction Per Second）：一次用户操作到服务器的返回结果，可以包含多个请求，通常包含事务执行。
* RT（Response Time）：响应时间。常见指标为AVG（平均响应）、95线和99线。
* AVG、95线、99线：99线表示按RT时间排序后，第99%个请求的性能范围。
  - 95线和99线与AVG线越接近，系统性能越平稳。
  - 99线可能是由于使用系统最频繁，数据最多，所以不能简单忽略。

数据库性能
* RT（Response Time）：响应时间。

GC性能
* 停顿时间（延迟）：停顿时间越短响应速度越高，用户体验越好。由于多线程GC，所以real应远低于user+sys。减小停顿时间的常用调优手段如下所示：
  * 调小GC启动阀值。
  * 增加并发标记线程数量。
  * G1算法调整停顿时间。
  * 增加年轻代大小。
  * 降低老年代提升次数。
  * 增加脏卡大小。
  * 减少机器上的I/O数量。
* 吞吐量：吞吐量越高越能更快完成任务。通常应保持在95%以上，对于后台批处理任务尤其如此。提高吞吐量的常用调优手段如下所示：
  * 增加GC开始阀值。
  * 减少并发标记线程数量。
  * 增加堆大小。
  * 采用G1算法时调整吞吐量大小。
* 停顿时间和吞吐量：两者互斥，对一个调优很可能会影响到另一个的性能，需要反复测试。
* 对象创建速率：创建速度越快越可能GC。
* STW和FGC：既要关注次数也要关注时长。
  - CMS算法STW发生在YGC、初始标记、最终标记、FGC阶段。
  - G1算法STW发生在YGC、初始标记、标记、Mixed、清理、FGC阶段。

### 性能限制

- 查询接口响应应控制在200ms以下，95线在500ms以下。
- 更新接口响应控制在600ms以下，95线在1s以下。
- 批处理控制在10分钟内。
- 复杂查询单表数据量控制在500万数据量以下。
- 单表主键查询控制在5ms以下。
- STW控制在250ms以内，YGC控制在10到20ms。

## 异常场景分析

{{< hint warning >}}
针对任何问题的处理口诀：先通报，后处理；先止损，后查因。
{{< /hint >}}

### GC性能

#### real > user + sys

- 后台磁盘I/O繁忙，引起GC日志暂停。
  - 减少机器上其它进程数量。减少日志数量。将日志记录在单独的HDD或SSD磁盘上。
- 没有足够的CPU资源来运行。
  - 减少机器上其它进程数量。

#### sys > user

- 操作系统出现问题。
  - 检查内存、磁盘、操作系统是否工作正常。
- 虚拟机没有足够资源运行。
- 内存不足，没有连续的页空间，引起页移动。
- 磁盘I/O压力过大。

#### GC原因一览

- Allocation Failure：年轻代没有足够空间分配内存。
  - 增加年轻代大小。
- Promotion Failure：年轻代对象无法提升到老年代。可能是由于老年代空间不足或存在大量碎片。
  + 降低老年代GC启动阀值。降低`CMSInitiatingOccupancyFraction`，设置`-XX:+UseCMSInitiatingOccupancyOnly`为true。
  + 降低标记整理启动阀值。
- Concurrent Mode Failure：并发回收失败。CMS无法及时完成对象的收集就会退化为并行收集器进行FGC。
  - 增加老年代大小。
  + 降低老年代GC启动阀值。降低`CMSInitiatingOccupancyFraction`，设置`-XX:UseCMSInitiatingOccupancyOnly`为true。
- Metadata GC Threshold：元空间GC。可能由于元空间太小或存在类加载器泄露（可能性很小）。
  - 设置 `-XX:MetaspaceSize` 来增加元空间大小。
- G1 Humongous Allocation：G1大对象分配。大对象指大小超过region一半大小的对象。频繁分配大对象会导致两个性能问题：大对象不足region空间的部分成为碎片，不会被使用；JDK 1.8u40之前大对象空间只在FGC时回收。
  - 设置 `-XX:G1HeapRegionSize` 增加region大小，避免大对象产生。
- G1 Evacuation Pause：G1疏散暂停，发生在region复制时。YGC和Mixed GC时会将存活对象从一组region复制到另一组region，如果此时空间不足就会触发FGC。
  - 内存方面只设置堆的最小或最大空间以及暂停时间，避免手动设置年轻代和老年代大小。
  - 如果问题仍然存在，增加堆大小。
  - 如果无法增加堆大小，并且标记周期开始过晚，可以减少 `-XX:InitiatingHeapOccupancyPercent` 的值。默认值为45%，表示内存利用达到45%时才开始标记循环。如果标记周期开始较早且未回收，则将 `-XX:InitiatingHeapOccupancyPercent` 阈值提高到默认值以上。
  - 设置 `-XX:ConcGCThreads` 来增加并发标记的线程数量。
  - 增加 `-XX:G1ReservePercent` 的值，使GC提前触发。默认值为10%，上限为50%，表示G1将始终保持10%的内存可用。

#### GC调优参数

调优策略

- 降低YGC频率：增大新生代空间，标记时间的成本低于复制的成本。
- 降低FGC频率：增加堆内存大小，减少大对象创建。
- 降低Biz日志产生，减少对GC日志创建的影响。

通用参数

- 最小堆最大堆设为一样，4G到8G间；
- 开启预分配内存；
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

- 延迟时间保存在200ms（默认）到500ms之间；
- 调整吞吐量，默认值12表示92%时间在应用上。20表示95%时间在应用上；
- 删除存活时间较长的重复字符串；
- 调整Mixed GC标记起始阀值，默认为45%；

### 接口性能

#### 应用长暂停

请求未在合理时间内返回。

原因分析：

- 对象创建速率过高，引起频繁GC。
  - 控制对象创建速率，可使用享元模式复用对象。
- 年轻代太小，对象过早提升到老年代。
  - 增加年轻代大小。
- 内存不足，发生磁盘I/O的进程切换。
  - 减少机器上其它进程数量，增加内存大小。
- real = user + sys，GC线程太少。
  - 增加GC线程数量。
- 后台磁盘I/O繁忙，引起GC日志暂停。
  - 减少机器上其它进程数量，GC日志写在单独磁盘上。
- 显式`System.gc()`采用了FGC。
  - 配置 ``-XX:+DisableExplicitGC` ` 禁用显式GC或者将显式GC换为CMS。
- 堆空间过大，每次扫描时间太长。
  - 控制堆大小在8G-16G。
- GC工作量分配不均匀。可能由于数据结构不支持并发扫描，并发分配失败等。
  - 配置 `-XX:+CMSScavengeBeforeRemark`

### 内存性能

#### 内存泄露

指对象无法被正确回收，随着时间流逝越来越多。

检查方法：

- jmap多次dump后，对比数据。泄露的对象会越来越多，不会被回收。