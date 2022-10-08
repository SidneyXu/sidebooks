---
weight: 1
title: Linux
---

# Linux

#### 网络请求分析

tcpdump

#### 上下文切换监控

- vnstat
- pidstat

以上两者都属于sysstat包，MAC不可用。

- wireshark：事件记录形式

- tcpdump

#### CPU相关

top 

按CPU利用率排序，按q退出交互

```shell
top -o cpu
```

## 内存相关

vmstat

```shell
vmstat <interval> <count>
```

![image](/images/devops/linux1.png)

- r：在运行队列中等待执行的进程数；一般每颗CPU的运行队列应控制在3以内；
- b：在等待I/O的进程数；此时CPU处于空闲状态，所以应越小越好；
- free：空闲的物理内存(kb)
- si：从磁盘交换到内存的swap页数量（kb/sec）；当内存满足时是不应该出现大量交换页的；
- so：从内存交换到磁盘的swap页数量（kb/sec）
- cs：每秒上下文切换次数
- us：用户进程的CPU时间占比
- sy：系统进程的CPU时间占比
- id：CPU空闲时间占比
- wa：等待I/O的CPU时间占比；过高证明瓶颈在I/O而不是CPU资源；

## 网络相关

netstat



## 磁盘相关

iostat

```shell
iostat <interval> <count>
```

-tx 报告每秒向终端读取和写入的字符数和CPU的详细信息

![image](/images/devops/linux2.png)


- %user：CPU处在用户模式下的时间百分比。
- %nice：CPU处在带NICE值的用户模式下的时间百分比。
- %system：CPU处在系统模式下的时间百分比。
- %iowait：CPU等待输入输出完成时间的百分比。过高表示I/O存在瓶颈；
- %steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
- %idle：CPU空闲时间百分比。如果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。
- Device：磁盘名称
- tps：每秒I/O请求数
- kb_read/s：每秒读取的block数。
- kb_read：读取的block总数。

## sar

### 基本概念

需安装sysstat包。查看使用方法如下：

```shell
man sar
```

### 查看CPU利用率

注意这里的CPU信息指向虚拟CPU，而不是物理机上的核数

```shell
sar -u <interval> <count>
```

![image](/images/devops/linux3.png)

- %user 用户级别CPU时间占比。
- %system 系统核心级别CPU时间占比。
- %iowait 等待I/O操作CPU时间占比；过高表示硬盘存在I/O瓶颈。
- %idle 空闲CPU时间占比；高但系统响应慢可能是CPU等待内存分配，需要加大内存；过低则CPU是瓶颈。

### 查看CPU负载

```shell
sar -q <interval> <count>
```

![image](/images/devops/linux4.png)

- runq-sz 运行队列的长度。
- plist-sz 进程和线程数的数量。
- ldavg-1 最近1分钟的CPU平均负载。

### 查看内存使用情况

```shell
sar -r <interval> <count>
```

![image](/images/devops/linux5.png)

- kbmemfree：空闲的物理内存大小
- kbmemused：使用中的物理内存大小
- %memused：物理内存使用率

### 查看I/O使用情况

```shell
sar -b  <interval> <count>
```

![image](/images/devops/linux6.png)

- tps：磁盘每秒I/O数量
- rtps：每秒读取I/O总数
- wtps：每秒写入I/O总数
- bread/s 每秒钟从磁盘读取的块总数
- bwrtn/s 每秒钟此写入到磁盘的块总数

### 查看磁盘使用情况

```shell
sar -p -d <interval> <count>
```

![image](/images/devops/linux7.png)

- tps：每秒I/O的传输总数
- rd_sec/s 每秒读取的扇区的总数
- wr_sec/s 每秒写入的扇区的 总数
- %util I/O请求占用的CPU百分比，值越高，说明I/O越慢