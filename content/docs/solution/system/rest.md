---
weight: 2
title: API设计
---

# API设计

## Restful接口

### 查询接口

按客户端类型返回

- GET /movies/100?format=full
- GET /movies/100?format=mobile

筛选字段

- GET /movies/100?fields=title,kind,rating
- GET /movies/100?exclude=actors
- GET /movies/100?includes=actors.name

条件过滤

- GET /movies?state=onshow&title=superman
- GET /movies?id=100,200

分页查询

- GET /movies?offset=20&limit=10
- GET /movies?last_id=10&limit=10

显示详情

- GET /movies/100

### 修改接口

- PUT /movies/100/avatar
	+ 创建或修改某个资源下的所有属性（通过payload指定）。
- PATCH /movies/100
	+ 只修改某个资源下的部分属性（通过payload指定）
- POST /movies/100/score?add=10
	+ 将资源的某个属性加10

## RPC接口

RPC接口指那些不涉及到资源修改的功能。

- GET /translate?from=en_US&to=zh_CN&text=Hello
- GET /calculate?x=100&y=200

## 接口版本号

- 版本号通过请求头version插入

