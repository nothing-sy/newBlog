---
title: 材质和纹理
date: 2024-03-25
categories:
 - babylonjs
---


>在场景概念中，网格Mesh、材质、纹理是相互关联的。 网格mesh相当于一个物体的结构，材质是物体的质地，比如金属材质，木质等等，不同的材质对光有不同的折射度，反射等特性。纹理则相当于在材质基础上的细节表现，比如一个木质桌子，木纹纹理就相当于在木质材质上添加了纹理细节。


```js
// 创建一个材质
var material = new BABYLON.StandardMaterial("material", scene);

// 设置材质的颜色，此处是漫反射，即对光的反射情况
material.diffuseColor = new BABYLON.Color3(1, 0, 0);

// 创建一个纹理
var texture = new BABYLON.Texture("textures/grass.png", scene);

// 将纹理赋给材质
material.diffuseTexture = texture;

// 创建一个mesh，这里创建了一个正方体
var mesh = BABYLON.MeshBuilder.CreateBox("box", {size: 2}, scene);

// 并将之前创建的材质赋给mesh
mesh.material = material;

```

### 多种材质

> 一个物体可能有多重材质组成，需要用到subMesh来重新定义mesh。

```js
/**获取材质和贴图 */
 function getMaterial(){
  const multip = new MultiMaterial('multipleMtl',scene)

  const baseMtl = new StandardMaterial('baseMtl',scene)
  baseMtl.diffuseColor = new Color3(1,0,0)
  // 创建基础材质
  const mtl = new StandardMaterial('mtl1', scene)
  mtl.diffuseTexture = new Texture("/zhuji/zhuji_start.jpg", scene);

  multip.subMaterials.push(mtl, baseMtl)

  return multip
 } 



 const box =  MeshBuilder.CreateBox('box',{
    size: 2
    },scene)
    box.position = new Vector3(0,0,0)
    box.subMeshes = [] // 将默认的子网格重置为空，一般情况下，一个网格只会有一个子网格，
    const verticesCount = box.getTotalVertices() // 从indexCount属性中可以知道顶点的数量，正方体为36，因此一面是6个顶点，按顺序为 背面，前面，左、右，顶部，底部。
    // 下面的参数，第一个参数为材质的索引，因为getMaterial中创建了两个子材质
    // 第二个参数为顶点索引，从0开始
    // 第三个参数为顶点数量
    // 第四个参数为起点的索引
    // 第五个参数为这个submesh要分配的顶点数量
    // 第六个参数则绑定某个要被切分submesh的mesh
    new SubMesh(0,0,verticesCount,24,6,box) // 顶部，24为起始点坐标 = 背面，前面+左右， 6为每个面的顶点数量
    new SubMesh(1,0,verticesCount,0,24,box) //  其余面
    new SubMesh(1,0,verticesCount,30,6,box) // 底部
    const multipleMtl = getMaterial()
    box.material = multipleMtl
```