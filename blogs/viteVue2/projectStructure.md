---
title: 目录结构及vite.config.js配置

date: 2022-3-24

categories:
 - viteVue2
---

>UI库的主要内容有两块，一块是组件库本身，一块是demo部分，用于示例的展示。因此需要区分两块内容分别为：packages和examples

```
目录结构：
	- examples 示例代码
		- main.js //主入口
		- App.vue //入口组件
		- assets 资源
		- router 路由
		- views 页面		
	- packages 组件库的包
	- public 静态资源
	- index.html
	- package.json
	- vite.config.js //配置库的打包
	- example.config.js//配置普通vue应用
```



因为项目中包含 组件库的内容和 demo示例的内容，因此分成了两个打包配置文件

vite.config.js默认为库的打包配置文件

example.config.js为示例的打包文件

```json
"scripts": {
    "dev": "vite --config example.config.js",
    "build-lib": "vite build",
    "build-demo": "vite build --config example.config.js",
    "preview": "vite preview"
  }
```

```js
//example.config.js
import {
  defineConfig
} from 'vite'
import {
  resolve
} from 'path'
import {
  createVuePlugin
} from 'vite-plugin-vue2' // vue2版本
import Markdown from 'vite-plugin-md' // 解析markdown文件
import hljs from 'highlight.js' //代码高亮

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
  ],
  resolve: {
    alias: {
      '@': resolve('examples'),
      '@pck': resolve('packages')
    }
  },
  build: {
    outDir: 'dist',
    rollupOptions: {
      input: './index.html'
    },
    chunkSizeWarningLimit: 1000
  },
  css: {
    postcss: {
      plugins: [{
        postcssPlugin: 'internal:charset-removal',
        AtRule: {
          charset: (atRule) => {
            if (atRule.name === 'charset') {
              atRule.remove()
            }
          }
        }
      }]
    }
  }
})

```

```js
//vite.config.js
import { defineConfig } from 'vite'
// import {resolve} from 'path'
import { createVuePlugin } from 'vite-plugin-vue2' // vue2版本

export default defineConfig({
  plugins: [createVuePlugin()],
  publicDir: false, //不需要将public目录的静态资源打包
  build: { 
    outDir: 'lib',//输出目录为lib
    lib: { //指定该打包为库打包
      entry: 'packages/index.js', //指定库的入口
      formats: ['es'],//指定打包方式，默认是es和umd ，这里只按es方式打包。
      fileName: (format) => `index.${format}.js`//打包后的名称,如果指定了formats为多种方式，则会打包成多个规范的文件
    }

  }
})

```

