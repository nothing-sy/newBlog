---
title: css效果Demo
date: 2023-04-14
categories:
 - css
---

## 下划线动效 background的用法

@[preview](@/.vuepress/vue-previews/Underline.vue)


## 水杯效果 clip-path属性裁剪
@[preview](@/.vuepress/vue-previews/Cup.vue)

## 字体交融 filter属性
@[preview](@/.vuepress/vue-previews/CrossWord.vue)

## Flip动画

Flip动画是一种实现思路并非某种技术。类似vue的内置组件transiton也是利用这种思路。例子： 当数组数据顺序改变希望有顺序改变的动画。
这种情况，使用单纯css是无法实现的。因此有了FLIP的思路。

F: first 记录元素起始位置
L: last 记录元素排序后终止位置
*上面两步之后我们就可以获取到，元素顺序变化后的偏差值，在浏览器还没有重新渲染的情况下，把dom顺序已经改变的元素通过设置偏移translate，还原到起始位置。
比如数组reverse，dom顺序已经倒叙，但是通过translate属性，让其保持在还未改变前的位置。即下面的步骤`invert`*
I: invert 反转元素到起始位置
P: play 播放动画 将之前设置的translate属性去除，还原排序后应该在的位置。

以上步骤加起来就会有一个动画效果，看例子。代码比较随意大致意思就这样

```vue
<script setup>
import {ref} from 'vue'
import {ElButton} from 'element-plus'
const cloneArray = arr => JSON.parse(JSON.stringify(arr))
const defaultList = Array.from({length: 5}, (item,index) => ({
  name:index,
  y: index  * 50 + index * 20,
}))

const list = ref(defaultList)


const revert = async () => {
  const reverList = cloneArray(defaultList).reverse()
  reverList.forEach((item,index) => {
    item.endY = index  * 50 + index * 20
    item.translateY = item.y - item.endY
    item.transform = true
  })
  list.value = reverList

 setTimeout(() => {
  list.value = reverList.map(item => {
    return {
      ...item,
      transform: false,
      transition: true
    }
  })
 }, 1e3)
 
  
}

</script>

<template>
  <div class="flex">
  <div class="item" v-for="item in list" :key="item.name"
  :style="{
    transform: `${item.transform ? `translateY(${item.translateY}px)` : 'translateY(0px)'}`
    ,transition: `${item.transition ? 'transform linear 1s': 'none'}`
  }"
  >
  {{ item.name }}
  </div>
</div>
  <ElButton type="primary" @click="revert">改变顺序</ElButton>
</template>

<style scoped>
.flex{
  display: flex;
  flex-direction: column;
  row-gap: 20px;
}
.item{
  width: 100px;
  height: 50px;
  line-height: 50px;
  background-color: aquamarine;
  border-radius: 10px;
}
</style>

```
