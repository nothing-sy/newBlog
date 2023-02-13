---
title: husky
date: 2023-02-01
categories:
 - 扩展
---

> husky 是一个辅助工具，允许你在git hooks周期内做一些额外的操作，比如配合eslint,style-lint,tsc等等工具做提交代码前的检查工作，防止团队成员因为各种原因，提交的代码不规范（比如eslint插件失效，人为关闭检测工具等）

官网： https://typicode.github.io/husky/#/?id=usage

根据官网安装husky，Create a hook。
至此基本的文件已经完成，项目下会有一个.husky目录
```md
 -.husky
  -_
    -.gitignore
    -husky.sh
  -pre-commit //这里就是安装husky以后创建的钩子，可以在里面写命令。加入你的项目中安装了eslit或者tsc工具，你的pre-commit内容可以为下面示例
```
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
npx eslint --fix //调用eslint 校验并修复
npx tsc //校验ts文件类型检查
```  

> 以上就是husky的基本用法，pre-commit文件将会在 git 提交命令前去执行你所编辑的指令。除此之外git还提供了各种钩子，具体可看[https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90]
再通过 npx husky add .husky/pre-commit "npm test" 命令创建新的钩子文件即可。

### 拓展
husky 默认会去校验【工作区】中所有的文件，这会导致检测特别的慢，而且有BUG，比如我使用pre-commit钩子，使用tsc做ts文件的检查，当我将错误的类型通过git add 将文件添加进暂存区，然后将工作区错误的类型改成正确的。 这是个执行git commit 则会因为工作区中错误的类型已经被修改成正确的，所以tsc的校验通过，但实际的暂存区，我们真正要commit的内容却是错误的。

因此我们需要一个工具，只检测在暂存区的内容。以下有两个工具
- lint-staged https://github.com/okonet/lint-staged
- nano-staged (lint-staged的简化版) https://github.com/usmanyunusov/nano-staged

上述两个工具都是专门解决这个问题的，此处只展示nano-staged的使用，lint-staged大同小异，参考官方网站即可

### nano-staged

根据官网安装nano-staged

#### usage

pre-commit文件
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
# git commit时利用nano-staged对暂存区的文件进行格式化及语法校验，详细配置可查看根目录下的.nano-staged.json文件（需安装依赖：husky、nano-staged）
npx nano-staged
```

配置nano-staged文件，支持多种形式的配置，如package.json字段，.nano-staged.json等文件配置

```json
{
  "src/**/*.{vue,scss,css}": "stylelint --fix",
  "src/**/*.{vue,ts,tsx}": [
    "eslint --fix",
    "bash -c vue-tsc"
  ]
}
```

> 配置文件规则很简单，key是要匹配的文件，value为要执行的命令
bash -c意思是将后面的命令作为字符串执行，相当于间接执行了vue-tes，这个用法是解决vue-tsc无法读取tsconfig.json文件配置加的，如果不加直接写`vue-tsc`则无法正确执行，也不会读取tsconfig.json ，如果不考虑ts的配置文件可以使用`vue-tsc --noEmit --skipLibCheck`

至此，nano-staged的基本配置已经介绍完毕，nano-staged会先将工作区的更改（还未git add）的内容，先stash储藏起来，等检查完暂存区的内容，确保没问题之后，再恢复储藏起来的修改，这样就能保证单纯装husky被绕开的问题
