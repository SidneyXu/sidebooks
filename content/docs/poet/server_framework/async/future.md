---
weight: 2
title: CompletableFuture
---

# CompletableFuture

## 基本概念

- Runnable：无返回值的函数。
- Supplier：有返回值的函数。
- Consumer：接受入参的函数。
- Function：接受入参且有返回值的函数。
- CompletionStage: CompletableFuture的父类。


## 基本使用

API接口列表

```java
CompletableFuture<Void> future1 = CompletableFuture.runAsync(runnable, executor);

CompletableFuture<Boolean> future2 = CompletableFuture.supplyAsync(() -> 1)
    .thenApply(integer -> integer == 1);
Boolean future2Res = future2.get();
```

以下常用方法中凡是方法名以Async作为后缀的都是异步方法：
- runAsync(runnable, executor) ：在线程池中执行Runnable。
- supplyAsync(supplier, executor）
- thenRun(runnable)：在当前线程中执行Runnable。 
- thenAccept(consumer)
- thenApply(function)
- exceptionally(function)：当发生异常时如何处理返回值。
- thenCompose(function, stage)：组合两个Future。
- CompletableFuture.allOf(stage..)：组合多个Future。
- get()：阻塞当前线程获得结果。

常见使用方法：通过supplyAsync()提供返回值；通过thenApply()或thenAccept()处理返回值；通过exceptionally()处理发生异常时的返回值。