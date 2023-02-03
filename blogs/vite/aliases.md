---
title: vite-aliases
date: 2023-01-06
categories:
 - vite
tags:
- vite
---

## 安装

>vite-aliases的作用是处理引入路径的别名

`npm i vite-aliases -D`

```js
import {
  ViteAliases
} from 'vite-aliases'
export default defineConfig( {
  resolve: {
    alias: [ //不止为何无法使用对象形式，服务会报无法遍历的错误
        {
          find: '@a',
          replacement: resolve( __dirname, './src/components/HelloWorld.vue' )
        }
      ]
},
plugins: [
  vue(),
  ViteAliases()
]
} )

```

> vite-aliases默认读取src目录下的所有目录名，如下所示

src =》 @
    assets =》 @assets
    components => @components
    pages
    store
    utils

在插件源码中，config文件配置的alias选项是优先于插件默认的配置的，比如config中已经配置了@src 指的为src目录， 则插件中@ 指向src目录这个配置会报错，因此避免与插件设置同一个目录，使用插件的情况下也不建议执行设置src目录



