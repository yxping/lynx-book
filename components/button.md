---
title: button 按钮
---

## button 按钮

该组件是按钮的基础组件。此控件完全由前端实现，可以通过`lynx-components`获取


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
|background  |  | #ff5777 | 组件背景色，默认值为:#ff5777,建议书写16进制        |
|foreground|  | #fff | button组件文本颜色 |
| text `required` | String |  | 按钮文本 |
|border-color | String | #ff5777 | button组件边框颜色 |
|font-size | Number | 26 | button文本大小 |
|font-weight | String | normal | button文本字体粗细 |
| onclick | Function |  | 触发后回调的click事件 |
| disabled | Boolean | true | 设置button组件是否可单击 |

### 事件

* `click`: 点击事件




### 例子

```html
<template>
  <view class="container" id="app">
    <view class="btn_container">
      <button
          class="btn"
          text="PRESS ME"
          background="#2198f2"
          foreground="#fff"
          :border-radius="2"
          :font-size="30"
          font-weight="bold"
          :onclick="clickHandler">
        </button>
      </view>
  </view>
</template>

<style>
  .btn {
    height: 100;
    flex: 1
  }

  .btn_container {
    width: 750;
    background-color: #f5f8f8;
    justify-content: center;
    align-items: center; 
    padding-left: 20;
    padding-right: 20;
    padding-top: 20;
    padding-bottom: 20
  }

  .container {
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>

<script>
import { Button } from 'lynx-components'
export default{
  components: {
    Button,
  },
  methods: {
    clickHandler (e) {
      console.log(e);
    }
  }
}
</script>
```

