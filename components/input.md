---
title: input 单行输入框
---

## input 输入框

用于表单文本输入 <del>支持双向绑定 v-model可以绑定input的值


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| type  | String | text | 用于设置input的类型,目前支持:text、number、Tel、password四种类型 |
| placeholder | String |  | input的默认文本 |
| maxlength| Number | 140 | input输入框能够输入的最大字符数 |
| change | Function |  | 输入框内容输入内容发生变化后的回调 |
| input | Function |  | 持续不断输入触发的回调 |
| focus | Function |  | 获得焦点后触发的回调 |
| blur | Function | | 失去焦点后触发的回调函数|
| <del> disabled | Boolean | false | input输入框是否可编辑|
| <del> defaultvalue | String |  | input的默认值 |
| <del> pcolor | String | #e3e3e3 | 表示placeholder的颜色 |
| auto-focus | Boolean | false | 表示是否获得焦点 |
| <del> hasclearbutton | Boolean | false | 表示是否有清除图标,主要给native那边用的，iOS支持，安卓暂不支持 |
| confirmt-ype | String | 'done' | android native键盘类型,可选的值有：send、search、next、go、done|
| confirm | Function | function(){} | 键盘回调 |
| cursorcolor | String | #000 | 光标颜色 |

### 接口

| API | 说明 |
|---|---|
| <del> getText | 获取文本内容 |
| <del> getFocus | 获取焦点方法  |
| <del> blurFocus | 失去焦点方法  |

