---
weight: 2
title: CI与代码管理
---

# CI与代码管理

## 分支管理

### 特性分支开发模式

1. 建立多个特性分支`feature/001`，在特性分支上进行研发。
2. 完成开发后，从主干上拉出发布分支`release/001`。
3. 将本次需要发布的特性分支全部合并到发布分支上。
4. 如果上线计划发生变动，临时不需要某个功能，可以从主干上拉出新的发布分支，进行合并。
5. 项目上线后，将发布分支合并到主干上。
6. 为主干添加标签，删除发布分支和特性分支。

### 主干开发模式

1. 在主干上拉出开发分支`dev/001`，在开发分支上进行研发。
2. 完成开发后，从主干上拉出发布分支`release/001`。
3. 将开发分支合并到发布分支上。
4. 项目上线后，将发布分支合并到主干上。
5. 为主干添加标签，删除发布分支和特性分支。

## 持续集成

工具一览
- Jenkins：CI工具，自动打包。
- Nexus：源代码仓库。
- Snoarqube：代码质量管理平台。
- FindBugs：静态代码分析工具。
- PMD：静态代码分析工具。
- Jacoco：测试覆盖率检查工具。
- CheckStyle：代码风格检查工具。

## 代码质量

如何保证代码质量？
- 设立代码规约；规范必要注释；
- 持续集成，代码合并前执行IDE Format、Checkstyle、FindBugs；
- 对复杂逻辑进行单元测试；
- Review代码和结对编程；
- 给予系统设计和思考时间，不随意压缩工期；