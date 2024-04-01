---
title: 坐标系
date: 2024-04-01
categories:
 - babylonjs
---

> 在babylon中有世界坐标系和局部坐标系，世界坐标系是左手定位，默认情况下是 大拇指向右，食指向上。即Z轴是指向屏幕方向的。且只会有一个世界坐标系
局部坐标系则是每个mesh都会存在，原点在中心，会随着网格的位置变更而变更。同时也可以通过调用transformnode和矩阵方式去改变原点在局部坐标系中的位置。此篇中我们不做详细介绍


## 关于ArcRotateCamera中，坐标系和旋转的关系

在ArcRotateCamera中，被观察的物体被设置在某个位置，而相机则围绕着这个物体进行旋转。

在这类相机中引入了alpha和beta两个参数，分别代表相机绕Y轴和X轴旋转的角度。详情看图：
![image](https://github.com/nothing-sy/newBlog/tree/master/blogs/babylonjs/imgs/arc.png?raw=true)



在babylon中没有内置的坐标系创建功能,需要通过其他方式创建。比如：

```js
// 通过创建线条系统来创建坐标系，参数就是XYZ三条线的起始点和终点，以及对应的颜色，此处对应为RGB三原色，
// 长的为正轴，短的为负值轴。

MeshBuilder.CreateLineSystem("Axis", {
      lines: [
        [new Vector3(100,0,0), new Vector3(-20,0,0)],
        [new Vector3(0,100,0), new Vector3(0,-20,0)],
        [new Vector3(0,0,100), new Vector3(0,0,-20)]
      ],
      colors: [
        [new Color4(1,0,0,1),new Color4(1,0,0,1)],
        [new Color4(0,1,0,1),new Color4(0,1,0,1)],
        [new Color4(0,0,1,1),new Color4(0,0,1,1)]
      ]
    },scene)
```