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
  context: path.resolve(__dirname, 'src'), //假设定义上下文环境为src目录
  //而不是 node进程目录（根目录）,则其他路径的配置需要响应变更
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
上面的配置中，由于定义了context的路径，则entry以及plugins的路径配置都是相对于src的目录,否则node进程目录为默认值的情况下应该配置为：
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
  //打包输出的文件名，默认会用[key]作为chunk的名称，这里对应 [key:] main => 生成 main.js，
  // [name]模板字符串可以参考官方文档output.filename提供的模板
  filename: '[name]', 
   //默认情况下，每个入口 chunk 保存了全部其用的模块(modules),这意味着如果你有多个入口文件，
   //并且都引用了某个模块代码，则这两个入口chunk会重复打包这部分代码。使用 dependOn 选项你可以与另一个入口 chunk 共享模块
  dependOn: ['vendor']
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