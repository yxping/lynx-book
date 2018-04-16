---
title: checkbox 多选
---

## checkbox 多选

该组件用于多选，支持双向绑定，此控件完全由前端实现，可以通过`lynx-components`获取


### 属性

| 属性名 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| change| Function |  | checkbox选中和取消的回调 |
| checkbox-group `required` |  |  | 用于包围一组checkbox,该标签上的bind-change表表示每一个子checkbox发生变化后的回调          |
| checked | Boolean | false | checkbox的选中状态 |
| disabled | Boolean | false | 禁止单击后的状态 |
| name | String |  | 用于标识checkbox |
| pic-array  `required` | Array |  | 有三个图片，分别是checkbox正常图片，单击后图片，禁止状态显示的图片 |
| width | Number | 48| 图片宽度 |
| height | Number | 48 | 图片高度 |

### 方法

  getCheckedName()方法在checkbox－group上可以获得选中的checkbox数组值



### 例子

```html
<template>
  <view class="container1" id="app">
    <checkbox-group :bind-change="changeHandler" ref="checkboxGroup" v-model="checkBoxValue">
      <view v-for="item of items">
        <checkbox class="checkbox"
          :name="item.name"
          :checked="item.checked"
          :pic-array="picArray"
          :disabled="item.disabled">
        </checkbox>
        <view class="viewText">
          <label>{{item.text}}</label>
        </view>
      </view>
    </checkbox-group>
   </view>
</template>
<style>
  .container1 {
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

  .checkbox {
    margin-left: 20;
  }
  .viewText {
    justify-content: center;
    align-items: center;
    margin-left: 20
  }
  .big {
    width: 50;
    height: 50;
  }
  .test2 {
    flex-direction: column
  }
</style>
<script>
const items =  [
  {text:"杭州",name:"HZ",checked:false,disabled:true},
  {text:"上海",name:"SH",checked:false,disabled:false},
  {text:"北京",name:"BJ",checked:true,disabled:false}
]

const picArray = [
  'assets/checkbox-check.png',
  'assets/checkbox-uncheck.png',
  'assets/checkbox-disable.png']

import {Checkbox,CheckboxGroup}  from 'lynx-components'

export default {
  components: {
    Checkbox,
    CheckboxGroup
  },
  data () {
    return {
      items,
      picArray,
      checkBoxValue: [],
    }
  },
  methods: {
    changeHandler (item) {
      var y = this.$refs.checkboxGroup.getCheckedName();
      console.log(this.checkBoxValue)
    }
  }
}
</script>
```



