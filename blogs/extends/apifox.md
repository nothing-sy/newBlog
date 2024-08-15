---
title: apifox 提高开发效率
date: 2024-08-15
categories:
 - 扩展
 - 工具
---

:::tip
Apifox集成了许多现有的工具，如Swagger、Postman、Mock、JMeter等，可以大大提高开发效率。

它支持API文档编写、API测试、API调试、API Mock等功能，可以帮助开发人员快速地构建和维护API。
该工具可以完全替代postman来辅助完成前端开发的工作
:::




# APIfox优化前端接口对接流程
在开发流程中，我们经常会遇到如下的问题。

1、与后端定义接口文档后，输出API文档， 前端需要根据API文档去生成typescript类型，并需要根据文档自定义一份mock数据
2、在开发过程中存在接口定义变更的情况，前端需要对应更新mock数据和接口类型
3、通过接口返回生成数据类型的时候也经常会出现因为文档中的变量实际类型为number，但是注释为字符串，导致转换类型时候变成string的问题存在。


为了解决以上的痛点，引入apifox，一个集成postman和swagger和其他工具的工具。

现有开发流程中，后端已经使用了swagger，前端可以利用导入swagger的URL，此处的URL是指直链（指向json数据的链接），并非swagger的UI 链接。以导入后端生成的接口，其中包含了一些运行实例，同时apifox也根据数据类型自动生成了mock的数据。

在调试接口的时候，我们可以在项目中设置一个代理，让接口请求链接到apifox启动的服务中，直接调用导入的接口。

有了正确的接口数据返回，再通过转换ts类型，则可以得到正确的数据类型。 当接口更新时，我们可以手动或自动更新接口文档，apifox也会为我们自动更新新的mock数据，保持一致性。

以下是大致的使用流程：

## 1、创建项目

在Apifox中创建项目，以此管理不同项目的接口。

![image](https://github.com/nothing-sy/newBlog/blob/master/assets/apifox/图片1.png?raw=true)

## 2、导入swagger url
![image](https://github.com/nothing-sy/newBlog/blob/master/assets/apifox/图片2.png?raw=true)

上图中是我们日常开发中使用到的swagger文档， 但这个url地址并非是我们需要的，我们需要的是path中对应的地址，不同的服务可能存在不同的地址。具体以实际为准。

Swagger url直连的地址一般是   path/v2/api-docs (swagger分为v1,v2,v3版本，目前使用的事v2版本)

![image](https://github.com/nothing-sy/newBlog/blob/master/assets/apifox/图片3.png?raw=true)

点击导入即可看到在[path]路径下部署的服务接口。 按需导入。

至此，接口已经可以使用。点击运行，即可返回mock生成的数据

![image](https://github.com/nothing-sy/newBlog/blob/master/assets/apifox/图片4.png?raw=true)


我们可以通过返回的数据，去生成typescript的类型



## 项目连接到apifox本地运行的mock

前面步骤中已经有了mock的接口并运行起来，当我们要调试接口的时候，需要定义一个规范，以etrm项目为例,  大部分接口以/app/ess 开头（具体以实际项目为准）， 所以我们定一个规范，对于apifox中的mock接口，以 /app/ess/mock开头，并重定向到apifox 的mock地址，该地址可以在配置中找到。

![image](https://github.com/nothing-sy/newBlog/blob/master/assets/apifox/图片5.png?raw=true)

> 下面是vue.config.js中的配置

```
proxy: {
        
        '/app/ess/mock': { // apifox mock数据专用
          target: 'http://127.0.0.1:4523/m1/4904252-4560785-default', // 这个地址指向了apifox的mock地址
          secure: false,
          changeOrigin: true,
          rewrite: (path) => path.replace(/\/app\/ess\/mock/, '')
        }
      }

```


配置完成后，即可直接调用运行中的mock。

![image](https://github.com/nothing-sy/newBlog/blob/master/assets/apifox/图片6.png?raw=true)
