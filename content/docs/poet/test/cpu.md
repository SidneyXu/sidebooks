---
weight: 3
title: CPU和线程的分析
---

# CPU和线程的分析

## top

top命令可以用来找出使CPU飙到100%的罪归祸首。

1、使用 `top` 命令查看进程列表

![alt](/images/test/top1.png)

以上输出结果中PID=382的进程占用CPU最高。

2、使用 `top -Hp <pid>` 查看进程内部的线程运行状态

![alt](/images/test/top2.png)

以上输出结果中PID=405的线程最为忙碌。

3、通过`printf %x <pid>`命令将线程ID转换为Java使用的16进制ID。

![alt](/images/test/top3.png)

4、然后就可以配合`jstack`命令找到对应的Java线程。

![alt](/images/test/top4.png)

{{< hint info >}}
为了查找方便，可以结合`grep`命令只返回后几行记录。

```
jstack <pid> | grep -A <n> <thread_name>
```
{{</ hint >}}


## async-profiler

官网：[https://github.com/jvm-profiling-tools/async-profiler](https://github.com/jvm-profiling-tools/async-profiler)

async-profilter采用了AsyncGetCallTrace API，是一款没有Bias问题的采样工具。支持CPU（默认）、alloc（内存）、锁、wall和itimer。

### CPU采样

开启/关闭采样

```shell
./profiler.sh start <pid>
./profiler.sh stop <pid>
```

在指定时间内自动采样

```shell
./profiler.sh -d <second> <pid>
```

生成Html格式火焰图文件。不指定文件时自动在控制台输出采样表格。

```sh
./profiler.sh -d <second> -f <html_filename> [--title <title_name>] <pid>
```

### CPU火焰图

火焰图本质是通过Linux的perf命令来生成的，该命令会返回CPU正在执行的函数名和调用栈。默认每秒采样99次，然后将CPU调用栈进行排列就产生了火焰图。

![image](/images/test/async-profiler1.png)

图表说明：
- 颜色无关紧要。
- y 轴表示调用栈，每一层都是一个函数。调用栈越深，火焰就越高，顶部就是正在执行的函数，下方都是它的父函数。
- x 轴表示抽样数，按字母顺序排列。宽度越宽，调用次数越多。
- 如果火焰图最上面存在大平顶，则表示该函数可能存在性能问题。

火焰图可以进行交互，在某一层点击，火焰图会水平放大，让该层占据所有宽度，显示详细信息。

