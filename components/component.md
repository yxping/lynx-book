---
title: 组件介绍
---

## 组件介绍

本文档将介绍框架组件相关的方方面面，包括组件的特点、组件的写法、基础组件，通过学习开发者能够迅速掌握Lynx相关组件并成功运行在业务中，现在开启我们的学习之旅吧。

### 组件特点

Lynx作为作为无线动态跨平台框架，其组件自然有别与其他类型的组件，其组件能够在Android、iOS运行。组件是视图层的基本组成单元。

### 组件写法

Lynx组件在开发上同前端框架Vue的写法类似，主要包括三部分：template、style、script，template表示组件的结构，style用来表示组件的样式，script用来表示组件的逻辑。组件在使用上是通过闭合标签的形式使用的，通过属性来修饰这个组件，形如下面:
```html
  <component property="value">
  </component>
```
**注意：所有组件与属性都是小写，以连字符-连接**

### 基础组件

框架为开发者提供了一系列基础组件，开发者可以通过组合这些基础组件进行快速开发。Lynx基础组件主要分为以下几大类:

**视图容器**

| 组件名 | 说明 |
|---|---|
| [body](./body.md) | 顶层容器|
| [view](./view.md) | 列表视图|
| [scrollview](./scrollview.md)  | 滚动视图 |
| [<del>listview-shadow]()  | shadow视图 |
| [viewstub](./viewstub.md)  | 排版视图 |
| [swiper](./swiper.md)  | page切换组件 |
| [slide](./slide.md)  | 拖动条 |

**基础内容**

| 组件名 | 说明 |
|---|---|
| [label](./label.md)  |  文本显示 |
| [img](./image.md)  |  图片显示 |

**界面切换**

| 组件名 | 说明 |
|---|---|
| [PageNavigator](./pagenavigator.md)  | 跳转链接使用组件|


**表单**

| 组件名 | 说明 |
|---|---|
| [button](./button.md)  |  按钮 |
| [input](./input.md)  |  用于表单文本输入 <del>支持双向绑定 v-model可以绑定input的值|
| [textarea](./textarea.md)  | 多行输入 |
| [checkbox](./checkbox.md)  | checkbox组件支持多选, <del>支持双向绑定 |
| [radio](./radio.md) | 用于多个选项中只能选中一个的情况,单选, <del>支持双向绑定</del> |

