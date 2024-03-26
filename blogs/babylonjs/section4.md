---
title: 交互和动画
date: 2024-03-26
categories:
 - babylonjs
---


# 交互
```js
// 创建一个简单的球体交互
var sphere = BABYLON.MeshBuilder.CreateSphere("sphere", { diameter: 2 }, scene);
sphere.position = new BABYLON.Vector3(0, 1, 0);
// 创建一个行为管理器并绑定到mesh
sphere.actionManager = new BABYLON.ActionManager(scene);
// 对这个mesh的行为管理器注册一个交互行为
// 此处ExecuteCodeAction为一个执行回调函数的行为，OnLeftPickTrigger为交互行为类型
// ActionManager包含了14种trigger方式，具体看官网
// 一下代码表示当鼠标左键点击球体时，球体缩放为1.5倍
sphere.actionManager.registerAction(new BABYLON.ExecuteCodeAction(BABYLON.ActionManager.OnLeftPickTrigger, function(evt) {
    sphere.scaling = new BABYLON.Vector3(1.5, 1.5, 1.5);
}
```


# 动画

```js
// 创建一个简单球体动画例子
var sphere = BABYLON.MeshBuilder.CreateSphere("sphere", { diameter: 2 }, scene);
sphere.position = new BABYLON.Vector3(0, 1, 0);

// 动画实际是一连串的关键帧，
const frameRate = 60 // 帧速率 ，一秒60帧
// position.x为球体的属性
const animate = new Animation('draw','position.z',frameRate,Animation.ANIMATIONTYPE_FLOAT,Animation.ANIMATIONLOOPMODE_CYCLE)
const keyFrames:any[] = [];
keyFrames.push({
  frame: 0,
  value: 0,
});

keyFrames.push({
  frame: frameRate, // 实际为时间t * 帧速率
  value: 2,
});

animate.setKeys(keyFrames);
sphere.animations.push(animate)
scene.beginAnimation(sphere,0,60,false,0.3)
```