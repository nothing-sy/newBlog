---
title: 模拟vite-plugin-mock-server

date: 2023-01-13

categories:
 - vite
---

## 作用

vite-plugin-mock-server 的安装和使用，请访问vite官网的插件文档，这里不做赘述
该插件的作用是，通过设置本地服务中间件，拦截请求，并返回自定义的数据。

简单的代码如下：
```js
export default () => {
  return {
    name: 'vite-MockPlugin', //自定义插件的名称
    configureServer( server ) { //配置本地服务器钩子
    //middlewares为服务器中间件，可以做一些处理，这里主要就是用来拦截请求，并将最终模拟的mock数据返回
      server.middlewares.use( ( req, res,next ) => {
        //req,res为请求和响应, next为将处理结果传递给下一个中间件（vite内部有许多中间件）
        if (req.url === '/api/test') { //假设拦截到了一个接口地址
          res.setHeader('Content-Type', 'application/json')
          const api = allApi.find(i => i.url === req.url) //在mock数据中找到对应的url地址，并调用response方法产生数据
          res.end(JSON.stringify(api.response()))// res.end为response的end方法，参考node的response，响应请求
        } else next() //如果没有拦截到请求，则将结果进行到下一个中间件
      } )
    }
  }
}
```

> 上面的简单示例代码为mock插件的大致原理， 拦截请求并做出响应