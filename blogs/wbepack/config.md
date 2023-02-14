---
title: webpack 5.x版本配置详解
date: 2023-02-13
categories:
 - webpack
tags:
  - webpack 5.x
---

:::tip
本节主要记录自己对`webpack5.X`版本官方配置项较重要或者较难理解的配置项做一些梳理。
:::

## 入口和上下文

### context 

> 基础目录，绝对路径，用于从配置中解析入口点(entry point)和 加载器(loader), 默认使用 Node.js 进程的当前工作目录

``` js
/**
 * 假设目录结构
 * root
 *  - src
 *    - index.js
 *    - vendor.js
 *  - index.html
 *  - webpack.config.js
 */

const path = require('path');

{
  context: path.resolve(__dirname, 'src'), //假设定义上下文环境为src目录而不是默认的node进程目录（根目录）,则其他路径的配置需要相应变更
  entry: {
    main: {
      dependOn: 'vendor',
      import : './index.js'
    },
    vendor: './vendor.js',
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'dist'),
  },
  plugins: [
    new htmlWebpackPlugin({
      template: '../index.html'
    })
  ]
}
```

:::tip
上面的配置中，由于定义了context的路径，则entry以及plugins的路径配置都是相对于src的目录,否则默认情况下应该配置为：
```js
{

  entry: {
    main: {
      dependOn: 'vendor',
      import : './src/index.js',
      filename: `[name][contenthash][ext]`

    },
    vendor: './src/vendor.js',
  },

  plugins: [
    new htmlWebpackPlugin({
      template: './index.html'
    })
  ]
}
```
:::


### entry 入口文件配置

> 一个需要考虑的规则：每个 HTML 页面都有一个入口起点。单页应用(SPA)：一个入口起点，多页应用(MPA)：多个入口起点

```js
{
  entry: { //entry有多种形式，可以是字符串，字符串数组，或者一个对象，具体的官方文档有说明
  main: { //此处主要讲解关于对象形式的描述符
  import: './src/main.js', //入口文件
  filename: '[name]', //打包输出的文件名，默认会用[key]作为chunk的名称，这里对应 [key:] main => 生成 main.js，[name]模板字符串可以参考官方文档output.filename提供的模板
  
  dependOn: ['vendor']//默认情况下，每个入口 chunk 保存了全部其用的模块(modules),这意味着如果你有多个入口文件，并且都引用了某个模块代码，则这两个入口chunk会重复打包这部分代码。使用 dependOn 选项你可以与另一个入口 chunk 共享模块
  },
  vendor: './src/vendor.js',

  }
}
```

:::tip
对于chunk的名称，配置文件有三个地方影响

1、如果entry是一个字符串或者数组，chunk的名称默认是main。

2、如果是多入口文件，配置的对象，则 key会作为chunk的名称。

3、2的基础上配置filename属性

4、如果指定了output.filename配置项，则此项会覆盖第2项



**如果同时写了好几个配置情况，优先级则为 3 > 4 > 2 | 1**，一般情况下不会对某个入口单独设置输出文件名
:::


## Mode 模式

> mode配置就三个： development | production | none
指定构建的版本是什么模式，不同模式，webpack内部做了一些配置优化， none则是不做任何优化，如果没有设置，webpack 会给 mode 的默认值设置为 production。
大多数情况，我们在开发时候使用development，生产环境使用production即可

*mode的配置方式除了可以配置mode属性，也可以在cli中使用`webpack --mode=development`*

:::tip
如果要根据 webpack.config.js 中的 mode 变量更改打包行为，则必须将配置导出为函数，而不是导出对象
```js
var config = {
  entry: './app.js',
  //...
};

module.exports = (env, argv) => {
  if (argv.mode === 'development') {
    config.devtool = 'source-map';
  }

  if (argv.mode === 'production') {
    //...
  }

  return config;
};
```
:::

## output 输出配置

>output 位于对象最顶级键(key)，包括了一组选项，指示 webpack 如何去输出、以及在哪里输出你的「bundle、asset 和其他你所打包或使用 webpack 载入的任何内容」


:::tip
在开始之前，我们应该理解chunk的概念，chunk是[块]的意思，源文件为一个个的模块，而chunk则是这些模块的集合。chunk分两种，一种是初始化chunk，一种是非初始化chunk

初始化chunk: initial(初始化) 是入口起点的 main chunk。此 chunk 包含为入口起点指定的所有模块及其依赖项

非初始chunk: non-initial 是可以延迟加载的块。可能会出现在使用 动态导入(dynamic imports) 或者 SplitChunksPlugin 时。 前者我们常常在框架路由中体现，比如vue,react的按需加载路由 () => import('xxx.vue')， 或者通过代码分割按需加载的chunk， 比如我们在引入一个文件 import('xxx.js')

通常情况下，一个入口文件会生成一个chunk，一个bundle文件，但是chunk和bundle不是一个概念，chunk是构建过程中的块。而bundle则是输出文件的结果。比如如果我们开启了sourcemap选项，则一个chunk会生成两个bundle，所以请勿搞混chunk和bundle的概念
:::


### assetModuleFilename
### asyncChunks
### chunkFilename
>非初始化chunk的文件名称，可以用占位符定义[name]...， 除设置chunkFilename以外，也可以在导入动态chunk的时候使用魔术注释 `import(/* webpackChunkName: "vendor" */'x.js')`。 魔术注释名称优先级 > chunkFilename

:::warning
由于webpack很多配置，部分配置可以达到相似的效果，因此理解基本的chunk概念对于使用这些配置有很好的帮助。比如使用动态导入方式，生成一个chunk，但是这种加载方式如果作为一个共享的代码块，会被执行多次，比如

::::code-group
:::code-group-item vendor.js
```js
export const vendor = '我是vendor'
console.log('我是vendor')
```
:::

:::code-group-item index.js
```js
import('./vendor').then(res => {
  console.log('我是index输出',res.vendor)
})

```
:::

:::code-group-item test.js
```js
import('./vendor').then(res => {
  console.log('我是test输出',res.vendor)
})

```
:::
::::

**上面的代码中,vendor.js会输出两次`我是vendor`**

如果使用别的方式生成chunk,比如单独设置一个入口为vendor, 且其他入口依赖于vendor，这样vendor.js一样会被单独作为一个chunk，且不会重复执行

:::


