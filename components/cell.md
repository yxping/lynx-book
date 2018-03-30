---
title: cell
---

## cell

`horizontal-listview` 按列排版时性能最佳，cell可以帮你拆分模块。使用时你可以把cell组件理解成一个从纵向变为横行的shadow组件，使用方式完全和shadow一样。

cell组件的父元素只能是horizontal-listview，不能隔代，不能嵌套多层cell。

### 样式和逻辑

`cell` 应该仅作布局使用，默认的 flex 布局方向是 `row`，请勿对其添加 css 样式，否则会产生不可预知的错误。
