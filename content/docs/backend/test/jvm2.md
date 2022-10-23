---
weight: 16
title: JVM常用设置
---

# JVM常用设置

## 通用设置

通用参数

```
# 应用
-Dspring.profiles.active=dev
-Duser.timezone=GMT+8
-Dfile.encoding=UTF-8

# JIT
-server

# 内存
## 最小/最大堆内存大小，一般设为容器内存的一半。
-Xms4g -Xmx4g 
## 元空间大小，一般设为256M-512M。
-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m
## 启动时预申请内存
-XX:+AlwaysPreTouch

# 日志
## 应写在单独的磁盘上。
## 使用/dev/shm 内存文件系统可以避免在高IO场景下写GC日志时被阻塞导致STW时间延长。
if [ -d /dev/shm/ ]; then
    GC_LOG_FILE=/dev/shm/gc-${APPID}.log
else
    GC_LOG_FILE=${LOGDIR}/gc-${APPID}.log
fi
## 打印GC日志
-Xloggc:${GC_LOG_FILE} 
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps 
-XX:+PrintHeapAtGC
-XX:+PrintPromotionFailure 
-XX:+PrintGCApplicationStoppedTime
```

可选参数

```
# 远程Debugger
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=7019

# 内存参数
## 线程栈内存大小。如果线程数较多，函数递归较少，可以调小该值节约内存，默认为1M。
-Xss256k
## 堆外内存的最大值默认约等于堆大小，可以显式将其设小，获得一个比较清晰的内存总量预估。
-XX:MaxDirectMemorySize=2g

# GC参数
## 并行GC线程数。
## 默认当CPU核数<=8时设为CPU核数。否则为(8+（CPU核数-8）＊5/8 )。一般改为CPU核数。
-XX:ParallelGCThreads=12 
## 并发标记线程数，默认为STW时并行线程数的1/4。一般改为CPU核数的一半。
-XX:ConcGCThreads=6
## YGC多少次后提升到老年代，降低可使对象尽快提升到老年代，提升YGC速度。
-XX:MaxTenuringThreshold=3
## 如果老年代较大，加大YGC时扫描的卡片来加快YGC速度，默认值256比较低。
-XX:+UnlockDiagnosticVMOptions -XX:ParGCCardsPerStrideChunk=1024

# 其它参数
## OOM时打印日志，在容器中运行时dump会影响到其它容器的运行。
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<file-path>
## 偏向锁
-XX:-UseBiasedLocking 
## 缓存整型
-XX:AutoBoxCacheMax=20000
## 禁止显式GC，使用堆外内存时不应设置该项
-XX:+DisableExplicitGC
## 软引用可存活大小，计算公式如下：
## last_gc_time - soft_ref_access_time <= freespace * SoftRefLRUPolicyMSPerMB
-XX:SoftRefLRUPolicyMSPerMB=0 
```

## CMS设置

```
# 内存
## 新生代和老年代比例，修改为1比1
-XX:NewRatio=1

# GC
## CMS启动阀值。
-XX:+UseConcMarkSweepGC 
-XX:CMSInitiatingOccupancyFraction=70
-XX:+UseCMSInitiatingOccupancyOnly
## System.gc()也使用CMS算法
-XX:+ExplicitGCInvokesConcurrent
## 并行处理引用对象
-XX:+ParallelRefProcEnabled 
-XX:+CMSParallelInitialMarkEnabled

# 可选参数
## 当CMS GC时间很长且受新生代存活对象数量影响时打开
## 会导致每次CMS GC与一次YGC连在一起执行，加大了STW时间。
-XX:+CMSScavengeBeforeRemark
## 如果永久代使用不会增长，关闭CMS时ClassUnloading，降低CMS GC时出现缓慢的几率
-XX:-CMSClassUnloadingEnabled
## 允许并行标记
-XX:+CMSParallelRemarkEnabled
## FGC几次后进行内存整理
-XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction=9  
## FGC前进行一次YGC
-XX:+ScavengeBeforeFullGC   

## 新生代大小
-XX:NewSize=1024m 
-XX:MaxNewSize=1024m 
## Survivor区比例，默认为8
-XX:SurvivorRatio=10 
-XX:+UseParNewGC 
## 解决1.8大对象分配时优化引起的GC BUG
-XX:-ReduceInitialCardMarks    
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager 
```

## G1设置

```
# 通用设置
## 采用G1收集器
-XX:+UseG1GC 
## Region大小，需要为2的倍数，范围在1MB到32MB间。
-XX:G1HeapRegionSize=8m
## 主要调优参数。GC预期最大暂停时间，默认为200ms。
-XX:MaxGCPauseMillis=200
## 主要调优参数。吞吐量，默认值12表示1/(1+12)=7.69%，即保证92.3%时间应用在程序上。
-XX:GCTimeRatio=12 
## 去除存活时间较长的重复字符串，JDK8u20以上可用。
-XX:+UseStringDeduplication
## 超过阀值时触发GC初始标记，默认为45%。
-XX:InitiatingHeapOccupancyPercent=45
## 年轻代最小值，默认为堆内存的5%。
-XX:G1NewSizePercent=5
## 年轻代最大值，默认为堆内存的60%。
-XX:G1MaxNewSizePercent=60
## Mixed GC时最大收集老年代数量，默认为10。
-XX:G1OldCSetRegionThresholdPercent=10
## 可保留内存大小，默认为10%，表示GC应尽可能使10%堆空间为未使用。
-XX:G1ReservePercent=10
```

## 