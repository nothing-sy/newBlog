---
title: 开发Chrome浏览器扩展插件
date: 2023-9-25
categories:
 - 扩展
---


# 如何开发一个谷歌浏览器插件

浏览器插件可以辅助我们浏览网页有更好的体验，我们能做的事情很多，谷歌浏览器给予了我们去拓展网页的能力，比如我们可以通过
给网站去除广告、获取视频网站的视频流并重新解析又或者给音频做一些列的音频处理、修改所有网页的默认样式。。。等等等等


因此，要开发一个谷歌浏览器插件，我们得去看看，谷歌提供的文档：
https://developer.chrome.com/docs/extensions/


我们以一个简单的例子： 获取视频网站连接并丢给第三方解析平台解析（第三方平台去了广告和vip的限制），此处第三方接口为： https://www.jsonplayer.com/

用法： https://jx.777jiexi.com/player/?url=https://www.ixigua.com/6801412550559793678 （将视频的url改成要解析的网址即可，这里不详细探讨这个第三方解析器的实现）

除了刚提到的jsonplayer还有很多其他的vip解析器：https://www.t312000.com/vipjiexi/



# 新建插件项目

包含以下几个步骤，具体的看谷歌开发文档就行
1、新增一个manifest.json文件，该文件配置了插件的版本，以及对应的文件目录。

manifest.json配置字段参考： https://developer.chrome.com/docs/extensions/mv3/manifest/

```
{
  "manifest_version": 3, //插件的版本（指谷歌插件开发本身的版本号，类似vue2,vue3）
  "name": "Reading time",//插件名称
  "version": "1.0",// 个人开发的插件版本号
  "description": "Add the reading time to Chrome Extension documentation articles",
  "content_scripts": [ //插件执行的内容scripts，还有很多其他字段，可以去官网查看
    {
      "js": ["scripts/content.js"],//执行的js文件路径
      "matches": [ //匹配的网址，必须是https://*这种形式，不能直接*号，这里指符合匹配的网址才会去执行这个js
        "https://developer.chrome.com/docs/extensions/*",
        "https://developer.chrome.com/docs/webstore/*"
      ]
    }
  ]
}
```


2、因为上面配置了一个`scripts/content.js`因此需要新建一个scripts目录和content.js文件，里面可以开始写代码了。比如。

```content.js
window.onload= () => {
console.log('文件加载了')
}
```

以上2点已经可以作为一个正式的插件了。打开浏览器扩展程序-开发模式。导入- 加载已解压的扩展程序，如果导入不成功就是语法有问题或者配置有问题，仔细检查下即可。

当我们访问matches匹配的网址时，就会这行这个js，打开控制台就可以看到输出的内容。


以上就是一个基本的完整流程。


### 解析视频的DEMO
在前面已经知道了基础的配置。那么现在要做一个解析视频的demo就简单了。
将matches匹配的网址全部改成，我们需要看的视频网址，比如https://iqiyi.com  或者其他视频网址


然后再`content.js`中写代码，比如简单的代码：
```content.js
window.onload = () => {

const url = window.location.href
window.open('https://jx.777jiexi.com/player/?url=' + url,'_blank') //这里直接把当前地址给到第三方解析器即可
}
```

以上就是当我们访问匹配的视频网址的时候，加载完毕就会跳转到解析地址。如果是视频就能观看了。

以上代码只是demo，并不好用，比如存在问题：
1、只能刷新网页才能跳转到解析网址
2、没有交互页面
3、因为匹配的网址没有精确匹配，所以可能是一些非某个视频播放网址，而是一个列表，那么跳转就会无效。
具体想更好用可以自己认真去写代码。这里只介绍基本的开发流程

### 正式的插件

以视频解析插件为思路，实现的插件： ![插件](https://github.com/nothing-sy/VideoChromeExtension/raw/main/img.png?/raw=true)

需求：当我需要解析视频的时候，由用户自己决定是否要解析视频，且不影响观看的页面本身的结构。

因此考虑使用popup的形式处理。当点击插件按钮的时候弹出一个页面，用户选择某个解析路径的时候，将当前活动页的url提交给第三方解析。

思路： 
1. 

[github仓库地址](https://github.com/nothing-sy/VideoChromeExtension)

该插件使用的是popup的形式，其manifest配置如下：

```json
//manifest.json
{
  "manifest_version": 3,
  "name": "VideoFree",
  "version": "1.0.0",
  "action": {
    "default_title": "选择解析地址",   
    "default_popup": "popup.html" //定义action的popup，即点击插件图标时弹窗的页面。该页面可以比作是一个新的浏览器窗口，其指向了popup.html文件
  },
  "permissions": [
    "tabs"//在调用chromeAPI的时候，某些字段的读取需要开启权限，此处开启tabs标签页权限，用于获取当前页签的url
  ]
}
```


```html
<!-- popup.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script  src="js/popup.js"></script>
  <style>
    .item{
      width: 100px;
      height: 20px;
      cursor: pointer;
      line-height: 20px;
      font-weight: bold;
    }
    .item:hover{
      background-color: cadetblue;
    }

  </style>
</head>
<body>
  <h2>解析路径</h2>
  <div id="wrapper">

  </div>
</body>
</html>
```

```js
//popup.js
let analysisUrlList = [{
      "url": "https://jx.aidouer.net/?url=",
      "name": "爱豆"
  }] //解析地址有很多，需要的话到原项目获取，部分解析可能已经失效，如果有收集到新的，可以自行添加修改
window.onload = () => {
  const wrapper = document.getElementById('wrapper')
  wrapper.append(...analysisUrlList.map((item,index) => {
    const el = document.createElement('div')
    el.setAttribute('class', 'item')
    el.setAttribute('data-url',item.url)
    el.innerText = item.name
    return el
  }))

  const list = document.querySelectorAll('.item')

  list.forEach(el => {
    el.addEventListener('click', async (e) => {
      const url = e.target.getAttribute('data-url');
        chrome.tabs.query({
          active: true }
          ,([tab]) => {
            //想要获取到tab.url这个参数，在manifest里面，必须配置permissions: tabs，否则无法获取
            //实际开发中请查找开发文档，会有说明某些数据是否需要开启权限
            window.open(url+tab.url, '_blnk');
        })
  
    })
  })

}
```

### 篡改猴（油猴）插件

篡改猴是谷歌浏览器的一个插件，它的主要功能是提供给用户搜索和管理脚本的功能。油猴是火狐的另一个插件，只是因为两者功能一致。所以人们习惯将两个东西混为一谈。

由于谷歌浏览器中如果要发布一个插件，是需要注册的，费用为5美元，因此篡改猴的目的就是让用户可以通过篡改猴，去使用更多的脚本（这些脚本的作者不需要注册为谷歌的开发，而是发布到其他平台，比如： greasy fork。），篡改猴就是在这些平台获取脚本，然后执行，以提供各种各样的功能。


### 如何编写篡改猴/油猴插件

根据油猴插件的编写文档： [地址](https://www.tampermonkey.net/documentation.php?locale=zh#meta:grant)

我将github上的谷歌插件仓库，另外改写成了油猴插件的版本，并放到了 [gist](https://gist.github.com/nothing-sy/b6fe077dd151a74d600207bf673f1123) 上面。文件后缀以.user.js 。  点击raw查看文件才会被油猴识别到。


可以通过在各种平台如greasy fork平台， userscipt.zone 或者 openusers、github/gist上面 发布插件