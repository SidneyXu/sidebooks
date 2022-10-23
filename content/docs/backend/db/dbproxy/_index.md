---
title: 数据库中间件
weight: 7
---

# 数据库中间件

## 知识点

。。。

## 最佳实践

MyBatis

- 数据库连接：`url: jdbc:mysql://localhost:3306/testDb?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8&useEffectedRows=true`
	+ useEffectedRows：返回SQL影响的行数。默认情况下返回的是匹配的行数，对于update语句来说无论是否更新成功，都会返回匹配到的行数。
