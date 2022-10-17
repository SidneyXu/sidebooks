---
weight: 3
title: Flex
---

# Flex

## 和Android Flex布局的对比

- flex：相当于 layout_weight，表示页面权重，用于填充多余部分
- justifyContent：子元素垂直对齐方向，可选值为：
  - flex-end：子元素靠底边对齐
  - flex-start
  - center
  - space-between：一个元素占顶部，一个元素占底部，其余元素平级间隔排列
  - space-around：每个元素前后都占据同样空间
- flexDirection：相当于 linearlayout 的排列方式，可选值为：column (默认)
- flexWrap：子元素占满时是否自动换行，可选值为：nowrap (默认), wrap
- alignItems：子元素水平对齐方向，可选值为：
  - flex-start：靠左排列
  - flex-end
  - stretch：默认拉伸
  - center 
- alignSelf：元素自身水平对齐方向，可选值同 alignItems    