---
title: Mesh的旋转rotation（涉及Local Space局部坐标系）
date: 2024-04-02
categories:
 - babylonjs
---


> 在babylonjs中存在全局的世界坐标系，以及mesh本身的局部/本地坐标系 (local space)。
而mesh的旋转就是以local space为基准，进行旋转操作。但是在3D空间中的旋转总是复杂的，需要考虑很多因素，例如旋转轴，旋转角度，旋转中心等。



:::tip
1. Where is the center of rotation? 旋转的中心在哪里
2. Are they applied in a clockwise or counter clockwise direction? 旋转是按顺时针还是逆时针
3. What is the frame of reference? // 旋转的参考坐标系是什么
4. In which order are they applied? // 他们旋转的顺序（x,y,z轴的旋转顺序）是怎样的
:::

:::tip
Answers:
1. The center of rotation is the center of the mesh. 旋转中心是mesh的中心
2. rotations are always counter clockwise when looking in the positive direction of the stated axis // 从起点往正方向观察，旋转总是逆时针的。
3. The frame of reference is the local space of the mesh. // 旋转的参考坐标系是mesh的局部坐标系
4. The order of rotations is  Y, X, Z. // 这个顺序取决于第3点。当参考坐标系为local的时候，顺序是！！！`Y，X，Z`。 **注意不是XYZ。**，先根据Y轴设置的弧度旋转，再根据X轴设置的弧度旋转，最后根据Z轴设置的弧度旋转。**每次旋转的时候，本地坐标系也是会一起旋转的**
:::

下面是参考图：

![image](https://github.com/nothing-sy/newBlog/blob/master/blogs/babylonjs/imgs/rotation2.png?raw=true)


## 详细图解分析

>旋转方法可以直接更改rotation或者调用addrotation方法

```js
// 第一种
box.rotation.x = Math.PI/2
box.rotation.y = Math.PI/2

// 第二种
box.rotation.x = Math.PI/2
box.rotation.y = Math.PI/2


// 第三种
box.addRotation(Math.PI/2,0,0).addRotation(0,Math.PI/2,0)
```

上面1,2两种虽然书写顺序不同，但是执行结果是一致的。因为babylon会优先按Y轴旋转，再按X轴和z旋转。

1. 当只执行`box.rotation.x = Math.PI / 2`的时候，看下local的改变。 原始的坐标轴可以参考世界坐标轴方向

![image](https://github.com/nothing-sy/newBlog/blob/master/blogs/babylonjs/imgs/rotation3.png?raw=true)

按上图展示，如果我希望三角形底边平行于 世界坐标轴的X轴。理论上只要绕local的Y轴旋转即可达到目的

我们从背面看：
![image](https://github.com/nothing-sy/newBlog/blob/master/blogs/babylonjs/imgs/rotation4.png?raw=true)

2. 执行`box.rotation.y = Math.PI / 2`，却得到了意料之外的结果
![image](https://github.com/nothing-sy/newBlog/blob/master/blogs/babylonjs/imgs/rotation5.png?raw=true)

> 因为上面说的，babylon始终先执行Y轴的旋转，再到X,Z。

那我们来看看，按照Y,X的顺序是怎样的：

3. 先重置代码，执行`box.rotation.y = Math.PI / 2`

![image](https://github.com/nothing-sy/newBlog/blob/master/blogs/babylonjs/imgs/rotation6.png?raw=true)

可以看到，根据Y轴逆时针旋转了90度。（记住所说的逆时针是从起点向正轴方向看的逆时针）

此时local已经改变，如果继续`box.rotation.x = Math.PI / 2`， 我们就会得到跟`第2点`图中所示结果。因此证明了，mesh的旋转确实是以local为参考，按Y,X,Z轴顺序改变的。

这种有违直觉的结果，在实际开发中，可能会导致一些问题。因此，还有一个办法，通过调用addRotation方法，可以确保按正确的顺序执行。`box.addRotation(Math.PI/2,0,0).addRotation(0,Math.PI/2,0)`。此时会先按照X轴旋转，再按照Y轴旋转。也就是在文章`第1点`执行完以后，按Y轴旋转，就得到了我们想要的结果。过程参考`第1点的两张图`

![image](https://github.com/nothing-sy/newBlog/blob/master/blogs/babylonjs/imgs/rotation7.png?raw=true)
