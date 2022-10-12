---
weight: 5
title: JVM的分析
---

# JVM的分析

## GC分析工具

- jstat
  - `jstat -gcutil <pid> <sec>` 查看应用启动到现在的GC情况
- jmap
  - `jmap -heap <pid>`
- GCeasy
- GCViewer

## 堆内存

### jmap

查看堆信息。

```shell
jmap -heap <pid>
```

导出堆信息，导出后的文件可以放入MAT来查看。

```shell
jmap -dump:format=b,file=<hprof_filename> <pid>
```

<img src="/Users/sidneyxu/dev/bitbucket/cafe/Side Project/capture/8、性能故障测试方法/image-20220803211312595.png" alt="image-20220803211312595" style="zoom: 50%;" />

## 线程栈

jstack：查看线程堆栈和死锁信息。