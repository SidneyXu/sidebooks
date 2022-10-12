---
weight: 1
title: Zuul
---

# Zuul

## 原理

![alt](/images/gateway/zuul.png)

四种Filter
- **pre**：路由前执行。可用于对请求进行预处理，比如鉴权、限流等。
- **route**：路由的具体行为，即远程调用的具体过程。
- **post**：路由结果返回或者异常信息发生后执行。可用于处理返回结果。
- **error**：路由生命周期任何阶段发生异常就会执行该Filter。可用于全局异常处理。
	+ 在 post Filter 抛错之前
		* pre、route Filter 没有抛错，此时会进入 ZuulException 的逻辑，打印堆栈信息，然后再返回 status = 500 的 ERROR 信息。
		* pre、route Filter 已有抛错，此时不会打印堆栈信息，直接返回status = 500 的 ERROR 信息。