---
title: checkbox 多选
---

## checkbox 多选

该组件用于多选，<del> 支持双向绑定


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| change| Function |  | checkbox选中和取消的回调 |
| <del> checkbox-group `required` |  |  | 用于包围一组checkbox,该标签上的bind-change表表示每一个子checkbox发生变化后的回调          |
| <del> checked| Boolean | false | checkbox的选中状态 |
| <del> disabled | Boolean | false | 禁止单击后的状态 |
| <del> name | String |  | 用于标识checkbox |
| <del> picArray  `required` | Array |  | 有三个图片，分别是checkbox正常图片，单击后图片，禁止状态显示的图片 |
| <del> width | Number | 48| 图片宽度 |
| <del> height | Number | 48 | 图片高度 |

### 方法

  <del> getCheckedName()方法在checkbox－group上可以获得选中的checkbox数组值

