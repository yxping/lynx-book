---
title: scrollview
---

## scrollview



拥有滚动条功能的view，不会回收元素，可以根据CSS布局来做X、Y轴滚动。

**注意：**使用时需要设定宽度，否则可能无法滚动。

### 属性

|属性名|类型|默认值|说明|
|---|---|---|---|
| fscroll-with-animation | Boolean | false | 为 true 时调用 `setScrollTop` 或者 `setScrollLeft` 会添加过渡动画 |
| scroll | Function | null | scrollview滚动时的回调函数 |
| scrollcancel | Function | null | scrollview滚动时手指离开屏幕的回调函数 |
| scrollstart | Function | null | scrollview开始滚动时的回调函数 |