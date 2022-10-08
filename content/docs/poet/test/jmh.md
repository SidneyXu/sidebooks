---
weight: 1
title: 基准测试
---

## JMH

测试代码执行效率如果碰到普通循环语句中。在其内部进行打印是没办法很好的测试出性能，因为会存在编译器优化，没办法实现性能测试需要达到的功能。
不过，JMH 也不能完美解决性能测试数据的偏差问题。它甚至会在每次运行的输出结果中打印上述语句，所以，JMH 的开发人员也给出了一个小忠告：我们开发人员不要轻信 JMH 的性能测试数据，不要基于这些数据乱下结论。



压测，重新进行压测,使用chaosblade添加网络丢包,延迟,并且观察grafana上的指标,来确定问题。

使用JMH

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

接下来可以运行mvn package命令，将编译好的 class 文件打包成 jar 包。生成的 jar 包同样位于target目录下，其名字为benchmarks.jar。jar 包里附带了一系列配置文件，如下所示：

打包生成的 jar 包可以直接运行。具体指令如下所示：

```shell
$ java -jar target/benchmarks.jar 
```

这里指的是 JMH 会 Fork 出一个新的 Java 虚拟机，来运行性能基准测试。
在这种情况下，通过运行更多的 Fork，并将每个 Java 虚拟机的性能测试结果平均起来，可以增强最终数据的可信度，使其误差更小。在 JMH 中，你可以通过@Fork注解来配置，具体如下述代码所示：

```java
@Fork(10)
public class MyBenchmark {
  ...
}
```

除了吞吐量之外，我们还可以输出其他格式的性能数据，例如运行一次操作的平均时间。具体的配置方法以及对应参数如下述代码以及下表所示：

```java
@BenchmarkMode(Mode.AverageTime)
public class MyBenchmark {
  ...
}
```

