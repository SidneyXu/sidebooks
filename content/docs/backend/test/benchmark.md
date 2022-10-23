---
weight: 1
title: 压力测试
---

# 压力测试

## 工具一览

- ab：Apache包中的测试工具。需要注意Mac自带的ab有并发数量限制，不能用于测试。
- JMeter：基于Java，可以在GUI界面上操作，无需编程基础。
- Gatling：基于Scala。
- locust：基于Python。

## Apache的ab工具

使用方法

```shell
ab -n <请求数> -c <并发数> <url>
```

举例：

```shell
ab -n 10000 -c 1000 -p 'params.txt' -T 'application/json' 'http://localhost:8080/api/clients/register'
```

输出结果：

```shell
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        
Server Hostname:        127.0.0.1
Server Port:            10808

Document Path:          /api/clients/register
Document Length:        436 bytes

Concurrency Level:      4000
Time taken for tests:   7.382 seconds
Complete requests:      10000
Failed requests:        578
   (Connect: 0, Receive: 0, Length: 578, Exceptions: 0)
Total transferred:      8119389 bytes
Total body sent:        5060000
HTML transferred:       4359389 bytes
Requests per second:    1354.63 [#/sec] (mean)
Time per request:       2952.841 [ms] (mean)
Time per request:       0.738 [ms] (mean, across all concurrent requests)
Transfer rate:          1074.10 [Kbytes/sec] received
                        669.38 kb/s sent
                        1743.47 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   37  48.3      1     167
Processing:    66 2416 805.2   2781    3296
Waiting:        6 2416 805.4   2781    3296
Total:        146 2453 760.8   2793    3296

Percentage of the requests served within a certain time (ms)
  50%   2793
  66%   2912
  75%   2966
  80%   3041
  90%   3065
  95%   3086
  98%   3118
  99%   3162
 100%   3296 (longest request)

```

参数说明

- Requests per second：吞吐率，指某个并发用户数下单位时间内处理的请求数；
- Time per request：上面的是用户平均请求等待时间，指处理完成所有请求数所花费的时间 /（总请求数 / 并发用户数）；
- Time per request：下面的是服务器平均请求处理时间，指处理完成所有请求数所花费的时间 / 总请求数；
- Percentage of the requests served within a certain time：每秒请求时间分布情况。指在整个请求中，每个请求的时间长度的分布情况，例如有 50% 的请求响应在 8ms 内，66% 的请求响应在 10ms 内，说明有 16% 的请求在 8ms~10ms 之间。

## JMeter

官网：[https://jmeter.apache.org/](https://jmeter.apache.org/)

## Gatling

官网：[http://gatling.io/](http://gatling.io/)

## locust

官网：[https://locust.io/](https://locust.io/)

