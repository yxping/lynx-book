---
title: view
---

## view

一般用来组成界面的某一块，与 `div` 相似。


---

一个使用click的示例

```html
<template>
  <view @click="clickHandle"></view>
</template>

<script>
export default {
  methods: {
    clickHandle() {
      console.log('clicked')
    }
  }
}
</script>
```