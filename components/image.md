---
title: img 图片
---

## img 图片

用来显示图片，使用时必须在CSS中指定宽高。

### 属性

|属性名|类型|默认值|说明|
|---|---|---|---|
| src `required` | String |  | 需要显示的图片链接，只支持线上链接 |
| placeholder | String |  | 占位图片链接，只支持线上链接 |
|<del> fixture | String | 无 | Web适用，设置是否忽略懒加载，默认不忽略。设置任意字符串生效。 |
|<del> recycle | String | 无 | Web适用，设置滚出屏时是否回收，默认不回收。设置任意字符串生效，fixture优先级更高。 |

可以通过ObjectFit来设置图片的拉伸方式。0, 1, 2分别对应FIT_XY，FIT_CENTER和CENTER_CROP的方式。