---
title: vite + vue2搭建UI库 目的和安装

date: 2022-3-24

categories:

 - viteVue2
---



### 目的

​	该项目的初衷是作为公司内部较通用的二次封装UI库，考虑到使用该库的人可能暂未学习vue3，所以依然选择vue2.x版本。

构建工具方面没有直接使用vue-cli的考虑是，vue-cli依赖于webpack。在构建方式和速度方面比较，vite的速度比webpack优秀，更方便本地调试和打包，因此选用vite + vue2搭建项目。

由于vite的构建默认使用了vue3，所以在创建vite项目后，需要进行额外的处理，以替换vue3



### 安装

1. 安装Vite，Vite 需要 [Node.js](https://nodejs.org/en/) 版本 >= 12.0.0。

   ```npm install vite@latest```

2. 创建项目

   ```npm init vite@latest my-vue-app --template vue```     **npm6.x和7.x版本指令有些区别，具体看[Vite官网](https://vitejs.cn/guide/#scaffolding-your-first-vite-project)** 

   以vue模板构建项目，配置默认为vue3， 创建项目后请勿直接执行npm install 去安装默认配置的依赖，因为要修改vue版本

3. 将vue3的相关默认配置，改成vue2 （**注意注释处为重点**）

```js
//package.json,下面只展示需要修改的部分，其余
{
  "dependencies": {
    "vue": "^2.6.11", //更改vue版本号为2.X
  },
  "devDependencies": {
    "vite": "^2.8.0",
    "vite-plugin-vue2": "^1.9.3",//Vite支持vue2
    "vue-template-compiler": "^2.6.14"//vue模板解析
  }
}

```

4. 安装package.json的依赖

   执行```npm install```，安装vue。vite-plugin-vue2,vue-template-compiler等依赖

5. 修改vite创建项目时，vue3的写法

````tex
//vite.config.js

vue3写法：
```
import vue from '@vitejs/plugin-vue'
export default defineConfig({
  plugins: [vue()]
})
```
改成：

import { createVuePlugin } from 'vite-plugin-vue2'
export default {
  plugins: [createVuePlugin()]
}
````



```tex
//main.js
//更改vue3版本写法 createApp 为 vue2的写法

import Vue from 'vue'
import App from './App.vue'

new Vue({
  render: h => h(App)
}).$mount('#app')
```

```text
//index.html
注意引入src的路径要指向main.js
```



除了上面的变更，还有默认vue文件的vue3 写法全部改成vue2即可。

