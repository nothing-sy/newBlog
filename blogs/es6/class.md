---
title: class this 和 super的指向问题
date: 2021-4-6
categories:
 - ES6
---

### class的this和super指向问题



- 普通类
  -  constructor内部， this指向类的实例   1
- 继承关系
  - 子类的constructor内部，this指向子类实例  2
  - super()调用父类的constructor函数。构造函数内部的this指向的是子类实例   3
  - super.xx 在普通方法中。super指向父类原型对象，即父类的prototype   4
  - super.xx在静态方法中super指向父类class本身，因此只能调用父类的静态方法，否则报错   5
  - super.xx在静态方法中，调用父类静态方法，父类静态方法中的this指向是子类   6