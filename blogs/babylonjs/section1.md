---
title: 理解3D场景的基础概念
date: 2024-03-22
categories:
 - babylonjs
---

## 三大概念

### 场景

就像在影棚拍摄一样，场景是所有物体的容器。我们可以布置背景，灯光，物体，以及拍摄用的相机等。
将所有的东西都和场景`scene`绑定，然后交给渲染器渲染

### 相机

相机是一个重要的概念，观察场景中的内容需要相机，相机决定了观察的角度，距离，以及观察的视角。可以比作为人眼。
babylonjs提供了多种相机，比如`FreeCamera`，`ArcRotateCamera`，`UniversalCamera`等
比如围绕某个物体旋转的相机`ArcRotateCamera`
比如跟随物体移动的相机`FollowCamera`
等等，这些相机的类型决定了如何观察场景内容

### 渲染器

当所有内容，包括相机，场景，灯光等准备就绪后，我们需要将它们渲染到屏幕上。
babylonjs通过创建`engine`，来实现渲染。

## 创建场景

```js
import * as BABYLON from 'babylonjs';
// 创建场景
var canvas = document.getElementById('renderCanvas');
var engine = new BABYLON.Engine(canvas, true);
var scene = new BABYLON.Scene(engine);
```

## 创建相机

```js
// 创建相机
var camera = new BABYLON.FreeCamera('camera1', new BABYLON.Vector3(0, 5, -10), scene);
camera.setTarget(BABYLON.Vector3.Zero());
camera.attachControl();
```

## 创建渲染器

```js
// 创建渲染器
engine.runRenderLoop(function() {
    scene.render();
});
```

