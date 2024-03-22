---
title: 光源、物体mesh
date: 2024-03-22
categories:
 - babylonjs
---


上一节中已经创建了 场景、相机、渲染器，这一节中我们来创建光源和物体。
没有光源物体是看不到的。因此需要创建光源，光源分为很多种，环境光、平行光、点光源、聚光灯等。


## 创建光源

```js
// 创建环境光源
var light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);
//设置位置并绑定到场景中
```

## 创建物体

```js
// 创建物体，在babylonjs中 内容对应的是Mesh，即以网格的形式来创建物体
// 下面创建了一个长宽高为2的立方体并绑定到场景中
var box = BABYLON.MeshBuilder.CreateBox('box', {size: 2}, scene);
```