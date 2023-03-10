---
title: webpack 5.x版本配置详解
date: 2023-02-13
categories:
 - webpack
tags:
  - webpack5.x
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
  runtime: 'mainRuntime', //这个入口chunk main的运行时环境，在这个chunk构建流程中，如果模块有重用的部分，会缓存起来，并适用缓存。 不同的chunk之间不共用这些缓存，除非配置了同样的runtime
  dependOn: ['vendor']//The entry points that the current entry point depends on. They must be loaded before this entry point is loaded， 说明这个入口chunk依赖于 指定的chunk， 必须先加载这个chunk， 而且功能类似于runtime，只不过这里指定的是入口chunk的名称，类似于 使用指定入口chunk的运行时环境
  },
  vendor: './src/vendor.js',

  }
}
```
:::warning
`runtime`配置，当没有这个运行时环境时会主动创建一个运行时环境， 而`dependOn`则不会主动创建， 两者不能同时使用
:::

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

**初始化chunk: initial(初始化) 是入口起点的 main chunk。此 chunk 包含为入口起点指定的所有模块及其依赖项**

**非初始chunk: non-initial 是可以延迟加载的块。可能会出现在使用 动态导入(dynamic imports) 或者 SplitChunksPlugin做代码分割出来的chunk**

通常情况下，一个入口文件会生成一个chunk，一个bundle文件，但是chunk和bundle不是一个概念，chunk是构建过程中的代码块。而bundle则是输出文件的结果。比如如果我们开启了sourcemap选项，则一个chunk会生成两个bundle，所以请勿搞混chunk和bundle的概念
:::


### assetModuleFilename（留空）
### asyncChunks
**创建按需加载的异步 chunk,默认为true，即按需加载的模块都默认创建一个异步chunk**

正常情况下我们是不需要去配置这个东西的，按需加载就创建即可，如果设置成false，则按需加载的代码还是会跟初始化chunk合在一起。

:::tip
webpack中有很多配置是互相影响/冲突的，其中配置的优先级也难以每个都进行实验。
比如`output.asyncChunks: false`，设置成按需导入的模块不作为新chunk，但如果你在下面中配置了这些选项，则
依然会作为新的chunk被打包。
```js
 optimization: {
    splitChunks: {
       chunks: 'all', //或者initial 
       minSize: 0
    }
  },
```
:::

:::tip
由于各种配置有影响，非必要/非常规的配置，建议以默认配置为主
:::


### chunkFilename
>非初始化chunk的文件名称，可以用占位符定义[name]...， 除设置chunkFilename以外，也可以在导入动态chunk的时候使用魔术注释 `import(/* webpackChunkName: "vendor" */'x.js')`。 魔术注释名称优先级 > chunkFilename

:::tip
谈到了chunk，那这里需要顺便提一下关于`optimization.splitChunks`,详情可以查看具体位置，这里只做简单介绍
默认情况下，webpack已经有了默认的代码分割策略（即只对动态导入的模块进行chunk分割），一般情况下对于initial chunk，父子chunk都会被打包到一起，而动态导入的模块，则会被作为新的chunk打包。 而splitChunks则是对现有默认分割策略做配置，其中 `splitChunks.chunks`就是设置分割模式： 'all' | 'initial' | 'async'， 表示是对不同类型的chunk做分割策略，如果选择all，则会两者都会处理，不管是动态导入还是静态导入。

指定了chunks分chunk模式后，还有其他的一些配置选项配合使用，比如设置了 'initial'之后，为什么N个入口文件公用导入的静态代码还是不会分出到新chunk，有可能是默认配置中`minSize`，指定了新chunk的大小，如果不满足条件，还是会被打包进原来的chunk中，不会单独分割

后续再详细介绍sliptChunks的内容，这里只简单讲解下

:::


### fileName

>此选项决定了每个输出 bundle 的名称。这些 bundle 将写入到 output.path 选项指定的目录下。
对于单个入口起点，filename 会是一个静态名称。

### globalObject

**此项配置适合用于输出类型为一个库的项目**

>当输出为 library 时，尤其是当 libraryTarget 为 'umd'时，此选项将决定使用哪个全局对象来挂载 library。为了使 UMD 构建在浏览器和 Node.js 上均可用，应将 output.globalObject 选项设置为 'this'。对于类似 web 的目标，默认为 self

### module

>以模块类型输出 JavaScript 文件。由于此功能还处于实验阶段，默认禁用。
当启用时，webpack 会在内部将 output.iife 设置为 false，将 output.scriptType 为 'module'，并将 terserOptions.module 设置为 true
如果你需要使用 webpack 构建一个库以供别人使用，当 output.module 为 true 时，一定要将 output.libraryTarget 设置为 'module'。


### publicPath(留空)

