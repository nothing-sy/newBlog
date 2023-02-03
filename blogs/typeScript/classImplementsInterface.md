---
title: class and interface
date: 2022-9-17
categories:
 - Typescript
---

### class 实现接口定义

接口可以作为一种约束，被class实现，接口只会去检查类的实例部分, typescript官方原话：
> 这里因为当一个类实现了一个接口时，只对其实例部分进行类型检查。 constructor存在于类的静态部分，所以不在检查的范围内

当然static标记的成员也属于静态部分，当你尝试在接口定义静态部分，会报错

```js
interface I {
  init ():void // you can't use static, you will get error static' modifier cannot appear on a type member
}

/**
 * Class 'F' incorrectly implements interface 'I'.
  Property 'init' is missing in type 'F'
*/
class F implements I { //报错，没有正确实现接口，当你尝试在interface加上static参数，接口会提示错误
  static init(){}
}
```

所以接口只能做到约束实例部分，而不能约束静态部分，如果需要约束静态成员（包括构造函数），我们可以使用间接的形式进行校验

```js
interface I {
    new(name:string,age:number):ClassInterface
}

interface ClassInterface {
    init(x:string):void
}

class Person implements ClassInterface {
    constructor(name:number){} //在这里故意写成name是number，这样当类传进去的时候就会被校验是否符合I接口
    init() {
        
    }
}

/**创建实例间接方法 */
function createInstance(c: I, name: string, age: number):ClassInterface {
    return new Person(age)
}

createInstance(Person,'name',14) //Error : Argument of type 'typeof Person' is not assignable to parameter of type 'I'
```

上面代码中，由于接口被类实现时无法检测静态部分内容，因此我们间接地通过参数校验结构的形式，去校验类本身的静态部分，这里指构造函数， 我们再Person中构造函数name写成了number属性，不符合接口I定义的string类型，因此会提示构造函数类型不匹配的错误。

这里有个有意思的点，假设Person构造函数为：`constructor(name:string){}` string类型，则编译器不会提示错误，为什么接口定义构造函数包含两个参数，但是校验的时候只有一个参数还是通过了检测？

因为在javascript中，尽管没有设置形参的个数，我们在调用的时候也可以无限制地往里面传入参数，或者 入参比形参个数少都是正常的情况。只不过拿到的参数是undefined
因此在ts中也保留了这一特性。 参数少的函数类型可以赋值给参数多的函数类型，比如：

```js
let a: (x: string, y: string) => void
let b: () => void
let c: (x:string) => string
a = b //正确
b = a //报错

a = c //正确  c返回string 也能通过void检测， 因为返回任何内容对void没有任何影响，
//但如果指定了返回string等类型（非any），则返回类型也必须一致(非any)

c=a //错误
```