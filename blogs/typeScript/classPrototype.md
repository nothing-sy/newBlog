---
title: class 成员和原型链
date: 2022-9-17
categories:
 - Typescript
---

### class 的定义，及其内部成员的关系

```js
class F {
  static staticMember =1
  name = 'name'
  common(){}
}

F.prototype.init = () => {}
F.prototype.age = 1

let res = new F()


```

在上述简单的例子中，包含了几种内容
首先我们要知道 class 定义的时候，包含了静态部分内容和实例部分内容
静态部分包含constructor构造函数和static标志的成员属性和方法。

静态成员我们可以直接通过类名调用，上面的例子就是`F.staticMember`

而`common`方法和`init`方法以及`age`属性都应该在F.prototype上，而`name`属性在 `res`的实例上

class的实例部分实际上包含的应该是 `name`属性和`common`方法，为什么`common`不是在类实例上，而是在F.prototype上。

因为在设计的时候，类作为构造函数工厂，其方法在调用的时候内部this是根据运行时指向而不同的，这意味着`common`方法不放在实例上，也能正确按照运行时的this指向去获取到正确的数据。但是成员变量确实实例独有的，否则所有实例的成员变量都会是同一个值。

总结下这章的内容，结合最上面的代码：
```js
//伪代码
F.prototype = {
  age:1,
  init:()=>{},
  common(){}
  constructor:{ //构造函数会包含静态部分
    staticMember:1
  }
}

let res = new F()
res = {
  name: 'name',
  __proto__: F.prototype
}

/*因此，当调用`res.init 或者res.common`方法，会去找F原型链上的方法
* 当找到了方法以后， 方法内部的this指向了 res实例，那么就能准确读取到实例内部的成员变量
*/
```





