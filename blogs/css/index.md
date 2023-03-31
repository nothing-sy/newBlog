---
title: css基础知识
date: 2023-03-17
categories:
 - css
---

:::tip
本节内容主要介绍一些基础知识，或者容易忽视或混淆的点
:::

## 选择器

### 选择器优先级

:::tip

>一个选择器的优先级可以说是由三个不同的值（或分量）相加，可以认为是百（ID）十（类）个（元素）——三位数的三个位数：
>
>ID：选择器中包含 ID 选择器则百位得一分。
>类：选择器中包含类选择器、属性选择器或者伪类则十位得一分。
>元素：选择器中包含元素、伪元素选择器则个位得一分。


*备注： 通用选择器（\*）、组合符（+、>、~、' '）和调整优先级的选择器（:where()）不会影响优先级。*

*否定（:not()）和任意匹配（:is()）伪类本身对优先级没有影响，但它们的参数则会带来影响。参数中，对优先级算法有贡献的参数的优先级的**最大值**将作为该伪类选择器的优先级*



| 选择器 | ID | 类 |元素|优先级|
|---|---|---|---|---|
|h1	|0	|0|	1	|0-0-1|
|h1 + p::first-letter|	0	|0	|3	|0-0-3|
|li > a[href*="en-US"] > .inline-warning|	0	|2	|2	|0-2-2|
|#identifier|	1|	0|	0	|1-0-0|
|button:not(#mainBtn, .cta)	|1	|0	|1	|1-0-1|

上述示例中 `button:not(#mainBtn, .cta)`之所以权重是是100 + 10 是因为not选择器只会根据传入参数的最大权重作为伪类选择器的优先级，因此实际权重是`button一个元素 + #mainBtn一个ID`
:::

覆盖声明的顺序
相互冲突的声明将按以下顺序应用，后一种声明将覆盖前一种声明：

用户代理样式表中的声明（例如，浏览器的默认样式，在没有设置其他样式时使用）。
用户样式表中的常规声明（由用户设置的自定义样式，比如浏览器提供的，允许用户更改字体大小等基本样式）。
作者样式表中的常规声明（这些是我们 web 开发人员设置的样式）。
作者样式表中的 !important 声明
用户样式表中的 !important 声明
用户代理样式表中的 !important 声明

:::tip
!important > 内联样式的权重 > 其他类型样式，无论有多少个ID选择器

所谓用户自定义样式指用户通过浏览器设置并改变的样式，目前只在浏览器找到一些字体相关的设置，至于其他复杂设置不同浏览器不同，早期的chrome和IE是允许提供一个custom.css去指定样式的。后面都取消了，旨在尽可能尊重开发者的设计
:::






### 全局选择器 *
除了常见的全局写法，还有子节点的全部选择，比如

```css
/**html下所有元素都会被选中 */
* {
  margin: 0;
}
/**div 下所有标签都会被选中 */
div * {

}


/**div下，所有第一个子元素都会被选中*/
div *:first-child{ /**等同于  div :first-child（中间有空格，表示div下的子元素） */

}

/**div 下所有不同类型元素 的第一个元素，比如div下有p和a标签，则所有p标签的第一个p标签，和所有a标签下的第一个a标签都匹配成功 */
div *:first-of-type {}


```

## 伪元素和伪类

### :first-child和:first-of-type

这两个伪元素很容易混淆，实际上都是有迹可循，`:first-child`表示指定选择器对象是其父节点的第一个子元素，`:first-of-type`表示指定选择器对象，是其父节点下（同一类对象的第一个子元素）

举例：

```html
<div>
<a>a1</a>
<a>a2</a>
<p>p1</p>
<p>p2</p>
</div>
```

```css
/**:first-child 指定了 div下的p标签是其所有兄弟元素中的第一个，div下第一个子元素是a1(也就是p的兄弟元素中的第一个)，
因此该规则没有匹配成功，不会有变化 */
div p:first-child {
  background:red;
}

/**指定 div下 p元素的第一个元素，即所有p元素中排第一个的元素，因此p1内容会被匹配上 */
div p:first-of-type {
  background:red;
}

```

## 浮动

所有的元素都可以浮动，只需要增加`float`样式，浮动的特点是，被设置浮动的元素，会脱离常规文档流，直到碰到其他浮动元素或者父容器的内容边为止。

### 浮动的影响
由于浮动元素脱离了常规文档流，所以其他元素会占据其原来的位置，但是内容区域会被浮动元素挡在外面。这是浮动元素的特性（典型文字围绕图像），如果要清除浮动元素对后续元素产生的影响，则需要使用clearfix技巧

### 清除浮动

下面一个html结构将用来介绍如何清除浮动

```html
<div class="wrapper">
<div class="float">浮动块</div> 
<p>我是紧随其后的P标签，文本块</p>
</div>
<div class="wrapper-after">我是wrapper后面的块</div>
```

 #### `clear`清除浮动

 >假设class为`.float`的块浮动起来了，则它脱离常规文档流，不占据高度，因此`P`标签会占据原来位置，且`P`标签的内容被`.float`块挤到`.float`外面。
 而且因为`float`脱离常规文档流，导致`.wrapper`块的高度由`P`标签撑开，即高度等于P标签高度，因此`.wrapper-after`也会跟着上移，这就是浮动的影响。


```html
<style type="text/css">
.float{
  width: 50px;
  height: 50px;
  float: left;
}
p{
  clear:left;
}

</style>
<div class="wrapper">
<div class="float">浮动块</div> 
<p>我是紧随其后的P标签，文本块</p>
</div>
<div class="wrapper-after">我是wrapper后面的块</div>
```


clear属性能够赋予某个元素清除浮动对后续元素的影响，我们只需要给不希望被浮动元素影响原来布局的元素，加上clear属性即可，详细参数查看MDN

#### `.wrapper`块添加伪元素::after 清除wrapper块以外的浮动

```html
<style type="text/css">
.float{
  width: 50px;
  height: 50px;
  float: left;
}
.wrapper::after{
content: "";
display:block;
clear: both;
}

</style>
<div class="wrapper">
<div class="float">浮动块</div> 
<p>我是紧随其后的P标签，文本块</p>
</div>
<div class="wrapper-after">我是wrapper后面的块</div>
```
这种方式可以清除wrapper块相邻元素的浮动影响，并不能清除内部影响


#### `overflow为非visible`

可是使用`overflow`设置成非visible以外的属性，可以使`.wrapper`块生成BFC，这样能正确计算wrapper内容的高度，使得对wrapper外的元素没有影响，但是内部内容还是会被影响

```html
<style type="text/css">
.float{
  width: 50px;
  height: 50px;
  float: left;
}
.wrapper{
overflow:auto;
}

</style>
<div class="wrapper">
<div class="float">浮动块</div> 
<p>我是紧随其后的P标签，文本块</p>
</div>
<div class="wrapper-after">我是wrapper后面的块</div>
```

#### `display: flow-root`

可以给`.wrapper`设置这个属性，相当于overflow的优化方案，对wrapper内部的元素还是有影响，只是清楚了`.wrapper`相邻元素浮动的影响

```html
<style type="text/css">
.float{
  width: 50px;
  height: 50px;
  float: left;
}
.wrapper{
display: flow-root;
}

</style>
<div class="wrapper">
<div class="float">浮动块</div> 
<p>我是紧随其后的P标签，文本块</p>
</div>
<div class="wrapper-after">我是wrapper后面的块</div>
```

## 盒子模型

### 盒子模型包含的内容
一个盒子模型包含外边距，内边距，边框，内容。

>在这里最好也解释下内部 和 外部 显示类型。如上所述，css 的 box 模型有一个外部显示类型，来决定盒子是块级还是内联。

同样盒模型还有**内部显示类型**，它决定了盒子内部元素是如何布局的。默认情况下是按照 **正常文档流**布局，也意味着它们和其他块元素以及内联元素一样 (如上所述).

但是，我们可以通过使用类似 flex 的 display 属性值来更改内部显示类型。如果设置 display: flex，在一个元素上，外部显示类型是 block，但是内部显示类型修改为 flex。该盒子的所有直接子元素都会成为 flex 元素，会根据弹性盒子（Flexbox）规则进行布局， 还有grid等布局

:::tip
也就是说display属性，对应了各种盒子的外部和内部显示类型。比如 `display:block`外部显示类型是 `块`，内部是普通的文档流布局。如果是`display:flex`则外部是`块`显示类型，内部是行内布局为`flex布局`，如果是`display:grid`外部显示为`块`类型，内部为`grid`布局， 还有其他可选值参考mdn
:::

### 标准盒子模型和IE盒子模型

盒子模型 包含标准盒模型和IE盒模型：

- 标准盒模型： 设置width和height实际是指content内容块的宽高
- IE盒模型：使用box-sizing: border-box ，则width和height是包含了内边距padding和边框border的宽高


## 定位

### 静态定位
`position`，所有的元素都有默认的定位方式，在正常的布局流中，元素的默认`position`为`static`

### 相对定位 `position:relative`
相对定位是相对于正常布局流中，位置的偏移。 使用属性`position:relative`， 然后通过`left/top/bottom/right`去移动元素位置。

:::tip
表现形式上，元素在原有布局流的位置上相对偏移了。但是占据的位置还是原来的位置，因此相邻元素位置不会改变
:::

### 绝对定位 `position:absolute`
绝对定位是相对于父级容器为非`position:static`的定位。

>如果所有的父元素都没有显式地定义 position 属性，那么所有的父元素默认情况下 position 属性都是 static。结果，绝对定位元素会被包含在初始块容器中。这个初始块容器有着和浏览器视口一样的尺寸，并且`<html>`元素也被包含在这个容器里面。简单来说，绝对定位元素会被放在`<html>`元素的外面，并且根据浏览器视口来定位

:::warning 注意
当设置为`position:absolute`时，你可以使用top、bottom和left、right 一组相反方向的属性来调节元素的大小，比如left:0; right:0 则宽度为父元素的内容宽度， 一般能预见绝对定位元素宽高的情况下，不使用这种形式设置大小
:::

### 固定定位 `position:fixed`

固定定位是相对于视口的定位，该位置不会随着文档滚动而改变位置

### 黏性定位 `position:sticky`
黏性定位是CSS3 的内容，相当于 相对定位 + 固定定位
表现形式为，当滚动条滚动时，黏性定位的元素到滚动元素视口的距离为设置的阈值，元素会固定在固定位置，如果小于这个阈值，则处于相对定位状态。

当指定一个元素为黏性定位时，其定位受两个因素影响：

1、其最近的，可滚动的祖先元素
2、其最近的，块级祖先元素

第1点，在设置为`position:sticky`时，配合`top/left/right/bottom`，指定该元素距离【最近可滚动祖先元素】视口阈值，当达到阈值的时候，元素则相对于滚动祖先元素固定定位，如果小于阈值则元素处在在某个位置（可能是初始位置，也可能是因为块级元素限制后的边界位置）

第2点，元素的固定定位不会 超过【最近块级祖先元素】的边界，即当元素滚动到块级祖先的边框，不再随着滚动条而移动，并停留在块级祖先元素边界。

下面是一个例子：

```html
 <style>

    .scroll-block{
      height: 500px;
      overflow: auto;
    }
    .wrapper-block{
      height: 300px;
      border: 1px solid #ccc;
    }
    .sticky-block{
      height: 150px;
      background:red;
      position: sticky;
      top: 20px;
    }
    .some-block{
  height: 600px;
}

    </style>


   
    <div class="scroll-block">
      <div class="some-block"></div>
    <div class="wrapper-block">
    <div class="sticky-block"></div>
  </div>
    <div class="some-block"></div>
    </div>

```


### z-index

在`position`不为`static`的时候，会有一个堆叠上下文产生。
目前`position`属性包含值：`relative/absolute/fixed/sticky`

:::danger 重点
一个元素的祖先元素如果已经定位了（即产生了堆叠上下文），则无论该祖先元素下有多深，其所有子元素的z-index层级与该祖先一致。因此，对该祖先元素下的定位元素设置z-index无效。（有意思的是，并不完全这样。当在孙元素下设置负数的时候，祖先的文本内容会显示出来，而不是被孙元素遮挡，但是背景等样式还是 孙元素 覆盖在 祖先元素上）

上面的情况意味着只有在两个不同的堆叠上下文中，才能进行`index`的比较。
:::

:::tip
当两个堆叠上下文 z-index一致的时候，顺序较后的元素，展示在上面。
:::


## 多列布局 `column-count`

此处多列布局是指`column-*`属性，通常使用`column-count`或者`column-width`使容器被计算分成多少列，详情查看MDN

## 响应式设计

### 媒体查询

查看详情：[媒体查询](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Media_Queries)

