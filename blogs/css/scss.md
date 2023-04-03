---
title: sass
date: 2023-04-03
categories:
 - css
 - sass
---

:::tip 前言
本章主要讲一些sass需要注意的点
:::

## 嵌套语法

sass的嵌套语法能减少很多重复的工作量，包含了以下N种形式：

### 普通嵌套
```scss
div {
  p {
    color: red;
  }
  a {
    color: blue;
  }
}

```

编译成： 
```css
div p {color:red}
div a {color: blue}
```

### 父选择器

```scss
div {
  color: red;
  &:hover{ // 被转成 div:hover
    color: blue;
  }
 .custom {
  &__left { //被转成 div .custom__left,直接把&符号理解成替换符号

  }
 }
  & & { //你也可以重复使用父选择器，会被转成： div div {color: gray}
    color: gray;
  }

  & > { // 编译成： div > div {} 
    div {

    }
  }

}
```

:::warning 注意
`&`选择符相当于替换，父级的选择器，当多层嵌套的时候，`&`表示N层父级。比如：
```scss
div {
  p {
    color:red;
    &:hover{}
  }
  .custom {
        &__left{}
  }
}

//编译成：
div p{color:red}
div p:hover{} // &等于 `div p`选择器
div p .custom__left {} //注意此处不会有层级.custom，而是直接.custom__left， 因为 &符号相当于`div p .custom`再拼接__left
```
:::

### 属性嵌套

除了选择器嵌套，属性也可以嵌套，诸如border-left/border-right等等
可以使用如下方式。这样就无需去记一些组合属性的有序和无序问题

```scss
  div {
    border:solid {
      left: 5px red;
      right: 5px blue;
    }
    margin: {
      left: 5px;
      right: 10px
    }
  }

//会编译成
```css
div {
  border: solid; //记住这个特殊点，但是有些属性因为css本身的问题，编译出来可能不是最佳实践导致样式无效，比如此处三个border样式是无法达到预期的
  border-left: 5px;
  border-right: 5px;

  margin-left: 5px;
  margin-right: 10px;
}

```

```


## mixin 混合器

混合器主要是用于处理一些多处使用的复杂的样式，无法单纯使用变量处理。
这里主要讲如何使用mixin

:::: code-group
::: code-group-item mixin.scss
```scss
@mixin custom-bg {
  color: red;
  background: blue;
}
```
:::
::: code-group-item index.vue
```vue
<style lang="scss" scope>
@import 'xxxxx.mixin.scss'
div {
  @include custom-bg;
}
</style>
```
:::

::::

:::warning 注意
使用mixin，要么在具体位置import，要么使用scss的data配置（详见官网），全局混入，否则会提示找不到这个混合器
:::