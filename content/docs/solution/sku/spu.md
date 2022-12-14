---
weight: 1
title: 商品系统
---

# 商品系统

## 数据建模

业务概念

- SPU：一种商品一个SPU。淘宝商品页面一个SPU一个页面，前台选择不同参数时不会发生跳转。
- SKU：SPU的物理形态，不同SKU是拥有不同可选参数的SPU。京东SPU页面实际由多个SKU聚合而成，选择不同参数会发生跳转。
- 商品分类：
  - 后台分类：后台管理用，一般最多设置三级。
  - 前台分类：用于前台展示，可任意设置，和后台分类为多对多关系。
- 属性项：不同分类拥有不同的属性项，每个分类由多个属性项组成。例如笔记本的属性项为触摸屏类型、分辨率、屏幕尺寸、操作系统、上市日期等。
- 属性值：属性项的值。
- 属性项分组：将属性项进行分组。例如笔记本的上市日期和操作系统属于基本参数分组，触摸屏类型、分辨率、屏幕尺寸属于屏幕参数分组。
- 品牌：商品的厂家。

表结构

- 商品基本信息表：商品的主副标题、价格、颜色等主要属性。每次修改商品信息时都需要保留历史信息。
- 商品参数表：商品的特征，不同分类的商品参数是完全不一样的。有以下几种实现方法：
  + 表中添加SPU列，将参数和SPU绑定。
  + 为不同分类的商品建立不同的商品参数表，如电脑参数表包含CPU型号和内存大小等字段，酒类参数表包含度数和产地。
  + 利用MongoDB等NoSQL数据库保存不同长度的参数列表。
- 商品介绍：包含大量的带格式文字、图片和视频。图片和视频保存在对象存储里。介绍文字和详情页一起静态化，保存在HTML中。

## 商品上架

业务流程

1. 商家填写商品详情，设置上架时间，支持表单的临时保存，此时商品状态为“待提交”。
2. 商家填写完毕后提交审核，此时商品的状态为“待审核”。
3. 后台人员进行审核，审核通过后将状态改为“待上架”。
4. 商家设置商品的上架时间，完成上架后状态改为“已上架”。

## 商品详情页

### 数据获取

商详页的数据通常来自多个接口，包括介绍文字，商品图片，评论，问答，库存，优惠券等。以下为常用实现方式：

介绍文字和商品图片等基本信息
- 主动获取：使用单一的汇总接口并行调用多个接口，组装后作为结果返回。
- 被动获取：详情系统通过消息队列接收变更，聚合后保存到Redis中。

评论和问答等辅助信息，需支持降级
- 异步获取：页面显示后再通过异步接口发起请求。
- 用户点击：默认不显示或只显示前几条，待用户点击相关按钮后再发起请求。


### 页面静态化

静态化可以节省服务器资源，利用 CDN 加速。商品介绍文字和图片可以静态化在页面中。而商品价格、促销信息等这些需要频繁变动的信息，可以由前端动态获取。

实现流程

1. 详情系统获取数据后生成详情页 HTML。
3. 通过 rsync 同步到其它机器。
4. Nginx 直接输出静态页。

## 商品搜索

### 搜索推荐

前缀匹配

用户每输入一个字都可能触发搜索，所以搜索推荐的请求量远高于搜索框中主动发起的搜索。ES针对这种情况提供了suggestion api，并提供的专门的数据结构应对搜索推荐，性能高于match，但只能做前缀匹配，并且需要结合pinyin分词器实现拼音提示中文。

非前缀匹配

非前缀匹配，需要使用Ngram并自定义分析器。

### 结果列表页

缓存热点搜索结果的第前N页。


