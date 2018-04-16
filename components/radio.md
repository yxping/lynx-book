---
title: radiobox 单选
---

## radio 单选

用于多个选项中只能选中一个的情况，单选。此控件完全由前端实现，可以通过`lynx-components`获取


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| checked| Boolean | false | radio的选中状态 |
| disabled | Boolean | false | 禁止单击后的状态 |
| name | String |  | 用于标识checkbox |
| pic-array  `required` | Array |  | 有三个图片，分别是radio正常图片，单击后图片，禁止状态显示的图片 |
| width | Number | 48| 图片宽度 |
| height | Number | 48 | 图片高度 |

### 例子

```html
<template>
  <view class="container" id="app">
    <radio-group :bind-change="changeHandle" ref="radioGroup">
      <template v-for="item in items">
        <radio  :checked="item.checked" :name="item.name" :pic-array="picArray" :disabled="item.disabled"></radio>
        <view class="itemLabel">
          <label>{{item.value}}</label>
        </view>
      </template>
    </radio-group>
  </view>
</template>
<style>

.container {
  flex-direction: column;
  background-color: white;
  width: 750
}
  .group {
    flex-direction: column;
  }
  .itemLabel {
    justify-content: center;
    align-items: center
  }
</style>
<script>
const items = [
    {
      name:'HZ',
      value:'杭州',
      checked:true,
      disabled:true
    },
    {
      name:'SH',
      value:'上海',
      checked:false,
      disabled:false
    },
    {
      name:'BJ',
      value:'北京',
      checked:false,
      disabled:false
    }
  ]

const picArray = [
  'assets/radio-uncheck.png',
  'assets/radio-checked.png',
  'assets/radio-disable.png']

import {Radio, RadioGroup} from 'lynx-components'

export default {
  components: {
    Radio,
    RadioGroup
  },
  data() {
    return {
      items,
      picArray
    }
  },
  methods: {
    changeHandle (item) {
      console.log(item.picArray)
      console.log(item.name)
    }
  }
}
</script>
```

