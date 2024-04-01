---
title: 创建坐标系
date: 2024-04-01
categories:
 - babylonjs
---


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