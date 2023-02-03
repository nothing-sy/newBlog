---
title: es6编程风格
date: 2021-4-20
categories:
 - ES6
---

### es6比较好的编程风格
- 尽可能用解构方式赋值
```js
const [a,b] = [1,2]
```
- Object对象尽量简写，尽可能不要添加新属性，如果需要，使用Object.assign

```js
const obj = {
a,
b
}
// 新增属性使用assign
Object.assign(obj,{p: ''})
```

- 如果只是为了使用key value，尽可能使用Map对象。自带遍历方法，而不是Object。

```js
const arr = [{key:'0:00',value:1},{key:'1:00',value:1}/* .....*/]
const map = new Map(arr.map(i => [i.key,i.value]))
map.forEach((value,key)) => {
    //to do 
})
```



- 尽量使用扩展符去拷贝数组

```js
const res = [...arr] //前提是arr值为非引用，否则为浅拷贝
```

- 总是用 Class，取代需要 prototype 的操作。因为 Class 的写法更简洁，更易于理解

>与function 的`prototype`的指向有所区别，class的`__proto__`和`prototype`属性
>
>（1）子类的`__proto__`属性，表示构造函数的继承，总是指向父类,即 class而非 类的prototype属性。
>
>（2）子类`prototype`属性的`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性。
>
>