---
title: eslint配置

date: 2022-3-29

categories:

 - viteVue2
---

## 安装

`npm i eslint -g`

在项目中执行`eslint --init`，根据自己的需求选择选项并生成 `.eslintrt`规则文件

node版本建议切换到 V14.x版本， 最新的eslint版本需要的插件依赖于对应版本

安装以下依赖：

```json
"devDependencies": {
    "eslint": "^7.12.1",
    "eslint-config-standard": "^16.0.3", //extends规则
    "eslint-plugin-import": "^2.25.4", //插件
    "eslint-plugin-node": "^11.1.0", 
    "eslint-plugin-promise": "^6.0.0",
    "eslint-plugin-vue": "^8.5.0",
  }
```



```js
//.eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true
  },
  extends: [
    'plugin:vue/essential',
    'standard'
  ],
  parser: 'vue-eslint-parser',//用vue的解析器
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module'
  },
  plugins: [
    'vue'
  ],
  rules: {
    'no-console': 'error',
    'no-debugger': 'error'
  }
}

```



