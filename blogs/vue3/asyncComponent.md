---
title: 异步加载组件
date: 2022-9-22
categories:
 - vue3
---

# asyncComponent

> vue3 提供了 defineAsyncComponent方法，用于页面需要渲染的时候，才对该组件内容进行加载
这与vue-router的路由动态加载不同，vue-router通过()=>import('xxx.vue')将整个路由页面动态加载，这里不要使用defineAsyncComponent，因为本身路由就是动态加载的，会出现问题。

路由组件动态加载后，其包含的所有子组件都会被下载，如果需要将这些子组件再异步加载，就需要用到 defineAsyncComponent， 基础用法如下：

```js
<script setup lang="ts">
import { defineAsyncComponent,Ref,ref } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/Common.vue')
)
let type: Ref<1|2> = ref(2)

/**当渲染条件达成的时候，common.vue文件才会从服务器加载 */
const change = () => {
type.value = type.value === 1 ? 2 : 1
}

</script>

<template>

  <div class="demo">
    <el-button @click="change">异步加载</el-button>
    <AsyncComp v-if="type === 1"></AsyncComp>
  </div>
</template>
```


> 注意，defineAsyncComponent接受的是一个返回promise的回调函数，所以我们一般结合import()来动态加载组件