---
weight: 1
title: Vert.x
---

# Vert.x

## 原理

- Vert.x Instances：Vert.x实例运行在自己的JVM中，是Verticle的运行环境。
- Verticle：Vert.x上的可执行单元。同一个Vert.x实例中的Verticle可以通过EventBus进行通信。
- Module：一个应用通常由多个Module组成。一个Module可以包含多个Verticle。
- Event Loops：Vert.x实例的内部的事件循环线程池，用于运行Verticle的代码。
- Worker Verticles：Worker对象运行在单独的Worker Pool中，可用于执行阻塞代码。
- Shared data：不可变对象，用于在Vert.x实例的不同Verticle间传递数据。