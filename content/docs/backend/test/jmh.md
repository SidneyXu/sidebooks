---
weight: 2
title: 基准测试
---

# JMH

## 基本概念

在测试代码执行效率时，由于存在编译器优化，普通代码调用是无法测试出真实性能的。而JMH的基准测试就可以解决编译器优化对测试结果造成的影响。不过，JMH 也不能完美解决性能测试数据的偏差问题，所以也不能轻信 JMH 的性能测试数据。

## 使用方法

生成 JMH 项目

```shell
$ mvn archetype:generate \
          -DinteractiveMode=false \
          -DarchetypeGroupId=org.openjdk.jmh \
          -DarchetypeArtifactId=jmh-java-benchmark-archetype \
          -DgroupId=org.sample \
          -DartifactId=test \
          -Dversion=1.21
$ cd test
src/main/org/sample/MyBenchmark.java
```

编译和运行 JMH 项目

```shell
$ mvn compile 
$ ls target/generated-sources/annotations/org/sample/generated/ MyBenchmark_jmhType.java MyBenchmark_jmhType_B1.java MyBenchmark_jmhType_B2.java MyBenchmark_jmhType_B3.java MyBenchmark_testMethod_jmhTest.java
```

运行`mvn package`命令，将编译好的 class 文件打包成 jar 包。生成的 jar 包同样位于target目录下，其名字为benchmarks.jar。jar 包里附带了一系列配置文件，可以直接运行。具体指令如下所示：

```shell
$ java -jar target/benchmarks.jar 
```

JMH 会 Fork 出一个新的 Java 虚拟机，来运行性能基准测试。
在这种情况下，通过运行更多的 Fork，并将每个 Java 虚拟机的性能测试结果平均起来，可以增强最终数据的可信度，使其误差更小。

```java
@Fork(10)
public class MyBenchmark {
  ...
}
```

除了吞吐量之外，还可以输出其他格式的性能数据，例如运行一次操作的平均时间。

```java
@BenchmarkMode(Mode.AverageTime)
public class MyBenchmark {
  ...
}
```

