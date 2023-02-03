---
title: markdown文件解析

date: 2022-3-29

categories:

 - viteVue2
---

### 使用markdown文件作为DEMO的示例文件

为了使得vite能够解析md文件，需要安装`vite-plugin-md`，（该插件依赖于markdown-it，里面涉及的细碎用法可以深入markdown-it做深入了解），

该插件同时兼容vue2和vue3，如果单纯为vue3服务，可以安装[vite-plugin-md2vue](https://github.com/WangXueZhi/vite-plugin-md2vue)

安装`vite-plugin-md`后，在vite.cofnig.js配置文件中使用

```js
import {
  createVuePlugin
} from 'vite-plugin-vue2' // vite兼容vue2版本
import Markdown from 'vite-plugin-md' // 解析markdown文件

export default defineConfig({
  plugins: [
    createVuePlugin({ include: [/\.vue$/, /\.md$/] }),
    Markdown({
      exposeFrontmatter: false, // 兼容vue2，definedExpose警告问题
      markdownItOptions: {
        html: true,
        linkify: true,
        typographer: true
      },
      markdownItSetup (md) { // 使用markdown-it的插件
        md.use(require('markdown-it-highlightjs'), { inline: true, hljs }) // 直接使用插件，不再自定义highlight风格
      }
    })
  ]}
```

### 路由文件

```js
{
  path: 'hdcButton',
  name: 'HdcButton',
  component: () => import('@/docs/hdcButton.md') //直接指向md文件，vite通过vite-plugin-md解析
}
```



#### 需解决问题

md文件解析后，发现代码块没有高亮展示。因为md只是根据markdown的结构被解析成了html，并不包含样式部分。为此，我们需要安装`highlight.js`来处理代码高亮部分。

`vite-plugin-md`的`markdownItOptions`属性中，可以配置highlight属性，如下：

```js
markdownItOptions: {
        html: true,
        linkify: true,
        typographer: true,
        highlight: function (str, lang) { // 回调函数，str是解析了markdown code block后的字符串，lang是代码块语言，即 markdown语法的```javascript/html/json 中的语言
    if (lang && hljs.getLanguage(lang)) {
      try {
        return hljs.highlight(str, { language: lang }).value; //返回字符串，此处是把md解析的字符串内容丢给highlight处理，并指定语言后返回，返回的字符串为被highlight处理过的html（各种包含class的html代码），然后另外引入highlight提供的众多样式文件中的一种风格，即可展示高亮。
      } catch (__) {}
    }

    return ''; // use external default escaping
  }
      }
```



`highlight`属性允许你自定义返回被解析后的markdown代码块内容，由于我们功能并不复杂，所以此处不使用这个属性自定义。而是使用`markdown-it-highlightjs`插件，该插件是符合markdown-it插件规则的插件，由于`vite-plugin-md`是基于`markdown-it`的处理，所以它还提供了函数，用于安装markdown-it的扩展插件。配置如上述中`vite-plugin-md`的markdownItSetup属性

```
markdownItSetup (md) { // 使用markdown-it的插件
        md.use(require('markdown-it-highlightjs'), { inline: true, hljs }) // 直接使用插件，不再自定义highlight风格,inline：true表示处理inline code的高亮代码。
      }
```



至此，差最后一步，引入其中一种代码高亮风格即可，highlight.js提供了一百多种样式，选择其中一种即可。

```js
//main.js
//在入口文件中引入样式文件
import 'highlight.js/styles/stackoverflow-dark.css' //stackoverflow黑暗风格
```



