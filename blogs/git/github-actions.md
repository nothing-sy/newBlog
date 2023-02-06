---
title: github actions自动构建博客并发布到github pages
date: 2022-2-4
categories:
 - git
---

:::tip
本节主要讲解，如何把一个博客项目，通过workflows 自动构建并发布到github pages

- 你将了解到 YAML的基础语法
- github actions 的基础使用
:::

## 需求

最近在升级vuepress-reco博客，从1.x 升级到 2.x版本，其生成的结构目录大概如下

```md
- .vuepress
  - public //资源目录
  - dist //构建目标目录
  - config.ts //vuepress的配置文件
- blogs // 博客目录
- docs // 文档目录
```

**github pages**

github pages的基本配置就是指定两个条件
1、 以哪个分支作为pages分支
2、 以哪个目录作为pages的根目录，只有两个选项 1: root根目录 2: /docs

如果使用比较笨的方法，可以在本地build 构建完成后把.vuepress/dist目录下的内容移到根目录的 docs目录，但是vuepress-reco2.x有个约定blogs和docs目录默认为博客和文档目录，所以会有冲突。

因此，希望能有一个完整的构建流程，省去我迁移目录步骤的方法

### workflows

workflow意思是一个工作流，它可以替你做任何在你自己电脑上能够完成的工作，github允许你在项目根目录下创建一个目录 .github/workflows/， 里面存放你需要的多个工作流文件，文件后缀为yml，使用yaml语法格式，形如
```yml
object
  name: x
  age: 17
```

表示为 object: {name: 'x', age: 17}的json格式

### github action

[github actions文档](https://docs.github.com/zh/actions/quickstart)

以下就是vuepress-reco 2.x 部署到github pages的配置

:::: code-group
::: code-group-item Demo
```yaml
name: build Blog for github pages #这个工作流的名称，不写默认以文件名作为名称
on: #工作监听
  push: #工作流触发器
    branches: #分支，可以是数组
    - master #上面加起来表示，监听master分支的push事件
jobs: #事务/作业 可以有多个作业，它不是数组而是一个对象
  publish_blog: # job的id，可以在上下文调用，加入有两个job，他们有顺序依赖，比如job2依赖job1，则可以在job2中配置needs: [job1]
    name: publish blog #job的名称，在构建过程中会显示正在执行的job的名称
    runs-on: ubuntu-latest # 这个job依赖什么环境，github-action上提供了一些环境，包括windows，因为windows最终有一个rsync的错误，在github actions的issue总建议使用其他系统，所以此处用了ubuntu最新版本
    steps: # 步骤，一个job可以分成N个步骤
    - name: Checkout # 步骤的名称，显示在工作流程中
      uses: actions/checkout@v3 # uses ，github actions提供了很多现有的配置，我们可以直接使用这些配置，这里的作用是切出分支，防止后续的操作
#     - name: setup node # 我原本以为是需要安装node的，但是实际上好像自带了这个，所以就注释了，如果发现有报错提示没有安装node则可以使用这个步骤
#       uses: actions/setup-node@v3.6.0
#       with:
#         node-version: 16.x # with参数是使用现成actions 需要带上的参数，具体在对应action中有文档说明
    - name: build blog
      shell: bash # 指定你要用什么来执行shell命令, bash支持github actions提供的各种平台
      run: | # run | 的写法表示后面的内容都作为字符串执行，此处就是正常的安装依赖和构建
        npm i
        npm run build
    - name: Deploy to GitHub Pages # 将对应目录打包成名为 github-pages的文件
      uses: actions/upload-pages-artifact@v1.0.7
      with:
        name: github-pages
        path: .vuepress/dist/

  # Deploy job
  deploy:
    # Add a dependency to the build job
    needs: publish_blog
    permissions:
      pages: write      # to deploy to Pages
      id-token: write   # to verify the deployment originates from an appropriate source

    # Deploy to the github-pages environment
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    # Specify runner + deployment step
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1.2.4 # 将上一步打包好的github-pages的文件，上传到githubpages

    #上述操作的 步骤 actions/upload-pages-artifact 和部署actions/deploy-pages@v1.2.4 可以使用以下第三方action一步完成，但是考虑到毕竟是官方的，就使用两个action处理
     # - name: Deploy to GitHub Pages #将构建好的代码部署到github pages
    #   uses: JamesIves/github-pages-deploy-action@v4.4.1 # 用的是第三方的action 而不是actions组织提供的。因为这个参数比较简单
    #   with:
    #   #  token: ${{ secrets.TOKEN }} # 如果你想跨仓库部署，你需要增加一个token已获得权限，此处为同一仓库，不需要
    #     folder: .vuepress/dist # 需要部署的文件目录，这个就是vuepress-reco的构建目录
    #   # branch: gh-pages #部署到哪个分支，这是默认值，这个action的原理就是将.vuepress/dist目录的内容，拷贝到创建的gh-pages分支的根目录，所以到时候会多出一个remote 分支，里面的代码就是 vuepress/dist目录下的内容
      
      
```
:::
::::

:::tip
上面的workflow走完以后，就已经多出了一个叫gh-pages的分支，根目录内容就是.vuepress/dist构建目录下的内容。
我们只需要再github pages中设置分支为gh-pages 且为/root根目录，即可

这样，当我们push代码到主分支，action监听到 on push事件，就会开始工作流。
自动构建并部署到gh-pages分支，github pages就完成了更新
:::
