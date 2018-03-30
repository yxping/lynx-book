---
title: textarea 多行输入框
---

## textarea 多行输入框

主要用于多行用于多行文本输入编辑，<del> 支持双向绑定


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| placeholder | String | 暗提示 | 表示textarea无内容时默认显示的内容          |
| maxlength| Number | 140 | maxlength属性表示textarea文本最多输入字符,最多输入140个字符 |
| change | Function |  | textarea输入内容发生变化后的回调 |
| input | Function |  | textarea持续不断输入触发的回调 |
| focus | Function |  | 获得焦点后触发的回调 |
| blur | Function | | 失去焦点后触发的回调函数|
| <del> value | String | false | 可选，String类型，textarea文本内容|
| <del>disabled | Boolean | false | textarea输入框是否可编辑|

