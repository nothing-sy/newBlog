---
title: GUI
date: 2024-03-29
categories:
 - babylonjs
---

## GUI

GUI指图形交互界面，在普通场景中除了渲染3D，可能还需要一些人机的界面交互，比如按钮，滑块等，babylonjs提供了GUI模块来支持这些功能。

>The Babylon.js GUI library is an extension you can use to generate interactive user interface. It is build on top of the DynamicTexture

GUI是依赖于DynamicTexture的，DynamicTexture是babylonjs提供的一种动态纹理，可以用来渲染UI界面。可以理解为一个画布，可以往上面绘制各种UI元素。

其中的类为`AdvancedDynamicTexture`，它具有两种类型，一种是全局的GUI，针对于一整个的场景，不会随摄像机的移动而移动，另一种是局部GUI，会随摄像机的移动而移动。

对于局部的GUI，它需要一个mesh作为它的父节点，这样它才会随着摄像机的移动而移动。

### 创建全局GUI

```js
var advancedTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI("UI",scene);
```

### 创建局部GUI

```js
var advancedTexture = GUI.AdvancedDynamicTexture.CreateForMesh(mesh);

```

:::tip
上述一个全局，一个局部的texture已经有了具体的媒介，一个是场景，一个是mesh。
场景的不需要额外的操作，而mesh则需要添加到场景才会生效。

做完上述之后，GUI还是没有内容的，因为仅仅是一个“画布”，一个容器，里面还没有放置内容，所以我们可以通过GUI模块下引入各种`Control`类来创建各种UI元素。
并添加到texture上
:::

### 创建按钮

```js
var button = GUI.Button.CreateSimpleButton("button", "Click me");
button.width = "150px";
button.height = "40px";
button.color = "white";
button.background = "green";
button.thickness = 0;
advancedTexture.addControl(button);
```

### 创建文本块

```js
const data = new TextBlock('data')
  data.fontFamily = "Helvetica";
  data.text = showPlaneData.value
  data.color = "white"
  data.fontSize = 18
  advancedTexture.addControl(data)
``` 

对于各种继承Control类的子类，都可以被GUI.Control.Container类通过addControl添加
