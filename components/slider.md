---
title: slider
---

## slider 拖动条

可以左右拖动的横条


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| min |  Integer | 0 | 设置滑动条的最小值 |
| max | Integer | 100 | 设置滑动条的最大值 |
| step| Integer | 0 | 设置滑动条的当前值 |
| value | Integer | true | 控制播放，为`true`时开始播放，`false`时停止播放 |
| active-line-color | String |  | 已拖动的颜色 |
| background-line-color | String |  | 未拖动的颜色 |
| change | Function | | 拖动条拖动时添加的回调|
