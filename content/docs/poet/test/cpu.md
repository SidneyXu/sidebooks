---
weight: 3
title: CPU和线程的分析
---

# CPU和线程的分析

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

