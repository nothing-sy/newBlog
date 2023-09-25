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


### 篡改猴（油猴）插件

篡改猴是谷歌浏览器的一个插件，它的主要功能是提供给用户搜索和管理脚本的功能。油猴是火狐的另一个插件，只是因为两者功能一致。所以人们习惯将两个东西混为一谈。

由于谷歌浏览器中如果要发布一个插件，是需要注册的，费用为5美元，因此篡改猴的目的就是让用户可以通过篡改猴，去使用更多的脚本（这些脚本的作者不需要注册为谷歌的开发，而是发布到其他平台，比如： greasy fork。），篡改猴就是在这些平台获取脚本，然后执行，以提供各种各样的功能。


### 如何发布greasy fork脚本

前面已经提到greasy fork是一个脚本平台，用户可以在上面发布和搜索脚本。 篡改猴可以通过安装greasy fork的脚本来提供各种能力。

那么如何发布greasy fork脚本呢。

需要到 https://greasyfork.org/zh-CN  ，登录用户后，点击用户名，进去可以看到发布脚本的方法。在此就不详细说明。

在编写篡改猴脚本的时候，篡改猴提供了一些开发文档，允许脚本的使用者使用一些方法或权限： 具体可以查看文档：https://www.tampermonkey.net/documentation.php。

根据文档编写的脚本符合篡改猴的标准，才能被正确安装、运行