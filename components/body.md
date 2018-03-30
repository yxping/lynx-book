---
title: body
---

## body


拥有滚动条功能的 view，并且在 Native 端会回收移除屏幕的元素提高性能，仅可以Y轴滚动。目前强制使用了 body 以获得更好体验，根元素必须是一个 body 标签，并且 `有且仅有` 一个。body的直接子节点高度越小（不大于屏幕大小），越有利于view回收，可以提高滚动性能

body标签在 Native 渲染模式实际上对应的一个 listview 元素，而不是 JS 中的 document.body 元素。


### 属性

#### <del> enablepull

* 类型: `boolean`，默认false

listview模式下，设置body是否需要下拉刷新
