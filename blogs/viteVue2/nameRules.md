---
title: 命名规则

date: 2022-3-25

categories:
 - viteVue2
---

>基本目录结构和组件加载方式定义好后，需要确定命名规则。

- packages目录下命名规则

```tex
- packages
	- hdcXXX （hdc开头的组件名 文件夹）
		- index.js //导出组件为插件
		- components //组件目录
			- index.vue //组件入口。 必须包含name属性，命名为hdcXXX与文件夹命名一致
```

- examples目录下命名规则

```tex
- examples
	- views
		- hdcButtonDemo //以组件名开头 + Demo结尾，表示该目录为 hdcXXX组件的DEMO示例组件目录
			- hdcButtonPrimary.vue //命名规范为hdcXXX组件+示例自定义名称。必须包含name属性
			- hdcButtonSuccess.vue
	- docs
		- hdcButton.md 文件名与packages下的命名规则保持一致
```

