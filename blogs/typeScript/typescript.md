---
title: Typescript 基础介绍
date: 2022-9-26
categories:
 - Typescript
---

# Typescript基础学习

## 目录

- 接口
  - 可选属性和只读属性
  - 字面量额外属性检测
  - 索引签名
  - 接口定义函数类型
- 类  
  - 类的类型
  - 类实现接口
  - 如何检测构造函数
  - 接口的继承
  - 接口的合并
  - 接口继承类（包含private和protected成员的类）
  - 重载
- 泛型
  - 泛型约束
  - 映射类型
  - extends
- 类型推导 infer
- 类型保护
  - 自定义类型保护
  - typeof 类型保护
- 类型兼容性
  - 子类型和兼容性
  - 变体
- 拓展
  - 可选函数和函数的参数this
  - vue组件引用ref的类型




## 接口

### 可选属性和只读属性
```ts
interface Person {
  name: string;
  age:number;
  money?:number;
  readonly sex:string;
}

let person: Person = {
  name: '某某',
  age: 17,
  sex: '男'
  // money非必须
}

person.sex = '女' //error sex is readonly property
```


### 字面量额外属性检测

> 该额外属性检测可以通过设置 tsconfig中编译项 suppressExcessPropertyErrors 取消

```ts
interface Person {
  name: string;
  age:number;
  readonly sex:string;
}


let person: Person = {
  name: '某某',
  age: 17,
  sex: '男',
  t: '' //error 字面量会做额外的属性检测，如果与接口不一致，编译器会认为这可能是一个错误
}

const obj = {
  name: '某某',
  age: 17,
  sex: '男',
  t: ''
}

let person1:Person = obj // correct 作为变量的时候不会做额外检测，只要满足接口属性即可
```


### 索引签名

```ts
interface Person {
  name: string;
  age: string;
  [key: number]: string; //必须确保 签名的索引类型 number的返回值与string类型的返回值一致，因为当通过索引访问的时候，number最终会转成string处理
  [key: string]: string;
}

//拓展- keyof any
  interface Person {
    [key: keyof any]: string; 
  }
// keyof 【索引类型查询操作符】 返回该对象所有 key 值组成的联合类型。
//  keyof any表示可用作对象索引的任何值的类型
// keyof any === number | string | symbol  //具体的联合类型根据ts版本以及配置不一致而不同[具体配置忘了]，主要是旧版本不支持symbol
```


### 接口定义函数类型

函数类型定义

```ts
interface F {
  (name: string): void; //函数的调用签名，无需具体的函数名称
}

let f: F = (name) => { }

```

```ts
interface F {
  (name: string): void; //函数的调用签名，无需具体的函数名称
  age:number
}

const ff = function(name){} //这里用function定义而不用箭头函数，是因为箭头函数无法作为构造函数
ff.age = 18 // 函数也是一个对象，为了满足F接口的定义，所以它包含了一个函数和一个age属性
let f: F = ff //此处如果ff.age属性不存在，则类型会报错

```

```ts
interface F {
  init:()=>void;
  call():void; //注意两者的差距，一个是定义了属性，一个是定义了函数
}

```


## 类

### 类的类型
> 定义一个类的时候，实际上我们定义了类的静态部分和实例部分
静态部分包括： 构造函数和static修饰符修饰的成员变量和方法
实例部分则是 非静态部分
静态部分的成员，我们并不需要实例化就可以直接使用

```ts
class Person {
  static someMember:any
  static eat(){}
  constructor(public name:string){} //简写，加上修饰符 ===  定义 name 并赋值  this.name === name
  born(){}

}
Person.prototype.age = 0 // 原型链的内容，属于实例部分
Person.sex = 'man' // 属于类的静态部分

//静态内容调用
Person.someMember
Person.eat()
let p = new Person('x')
//实例化内容调用
p.born()

//该类的结构【伪代码】
Person.prototype = {
  constructor: {
    someMember,
    eat,
    sex
  },
  born,
  age
}

let p = new Person()
p = {
  name: 'x', //实例部分
  __proto__: Person.prototype
}

```


### 类实现接口

> 接口是一种约束， 类实现接口意味着接口定义的成员属性，都应该被类实现

```ts
interface I {
  init(): void;
  name:string
}

class Person implements I {
  init(){}
  name = 'xx'
}
```

> 接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员

这意味着，接口无法约束类的静态部分（构造函数和static修饰）和私有部分（protected 和private）

```ts
interface I {
  init(): void;
  name: string;
  age: number; // 无法在接口使用private static protected new修饰符，因此它只能是public属性
}

class Person implements I {
  init(){}
  name = 'xx'
  private age = 18 // error 没有实现接口定义的age (public)
}
```

如果我想要检测构造函数呢？

> 接口对类的检测限制在公共部分，但是在正常的类型校验中并不会有这么个规则，那我们只要将构造函数作为正常的函数校验即可

```ts
interface ConsturctorInterface {
  new (name:string):ClassInterface //返回一个接口类型，是为了让返回值可以向下转型,我们也可以定义一个泛型，让接口正确返回构造函数的实例类型
}

interface ClassInterface {
  init(): void;
  age: number;
  name: string;
}


class Person implements ClassInterface {
  age: number;

  constructor(public name: string) { //构造函数简写 public name === this.name = name
  }

  init(){}
}

class Person2 implements ClassInterface {
  age: number;
  name: string;
  constructor(name: number) {
  }

  init(){}
}

function createdPerson(c: ConsturctorInterface,name:string): ClassInterface{
  return new c(name)
}

let person = createdPerson(Person, 'name') as Person
let person2= createdPerson(Person2,'person2') //error 构造函数name类型应该为number
```


### 接口的继承

```ts
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```


### 接口的合并(声明合并)
>“声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。 合并后的声明同时拥有原先两个声明的特性。 任何数量的声明都可被合并；不局限于两个声明
```ts
interface I {
  a:number
}

interface I {
  b:number
}

let res: I = {
  a: 1,
  b:1
}


```

出于严谨的考虑，我们应该使用继承或者直接写完整而不应该使用接口合并。详细的声明合并内容不再这里探讨



### 接口继承类

> 当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。 接口同样会继承到类的private和protected成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}


// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}
```


### 函数重载

> 为同一个函数提供多个函数类型定义来进行函数重载。 编译器会根据这个列表去处理函数的调用。 下面我们来重载 pickCard函数

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];
//不同的函数类型定义
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };

//函数的定义
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

### 类的函数重载

```ts
class F implements I {
  //类里面写函数类型定义，相当于一种占位符
  init(x: number): number;
  init(x: string): string;
  init(value:any){
    if(typeof value === 'number') {
      //do something
    }
    if(typeof value === 'string') {
      //do something
    }
    
  }
}

let f = new F()
let r = f.init(1)
let rr = f.init('')
```

> 我们应该考虑什么时候使用重载，而不是只要参数类型不一致，就使用
比如可以考虑可选参数，多个参数，或者泛型处理


## 泛型
> 当我们不确定接收类型的时候，我们可以使用类型变量来定义，也就是我们所说的泛型，可以有泛型函数，泛型类型别名，接口泛型，类泛型等等

```ts
function test<T>(value:T):T{return value}

type Alias<T> = T

interface I<T>{
  init(value:T):T
}

class C<T>{
  name:T
}
```

### 泛型约束 extends
>泛型约束顾名思义是对泛型变量的约束，使用extends关键字

```ts
interface I {
  name:string
}

function test<T extends I>(value:T):T {
  return value
}

class C {
  name:string
}

test({name:'x',age:1}) // 返回 入参类型  {name:string,age:number}

test(C) //返回typeof C 即 C 类的静态类型而不是 C类型（实例类型）
```

上述泛型变量T 被 I 接口类型所约束，则test方法入参必须符合I接口类型的数据


### 联合类型的extends用法

联合类型的“继承”或者说“子集”，看起来比较不直观，比如 

`'a' | 'b'` 和 `'a'` ，其中 `'a'`类型是 `'a' | 'b'`类型的子集，所以 `'a' extends 'a' | 'b'`，可能对继承的概念我们会优先想到[子类会比父类更多内容，那么看上去'a'|'b' 比'a'类型内容更多]， 其实这个直观感受是错误的。 应该理解为**子类在父类基础上更具体，更细致**，因此在联合类型上看`'a'`类型比`'a'|'b'`类型更加具体。因此才会有`'a' extends 'a' | 'b'`的结论。ts也确实是这样的。

有意思的是ts是如何去处理 联合类型直接的继承的。可以看内置的类型 
```ts
type Exclude<T,K> = T extends K ? never : T

type R:Exclude<'a'|'b'|'c'|'d', 'a' | 'b'> // type R === 'c' | 'd'

// 其实T extends K 将类型拆解处理了，情况如下：
// 'a' extends 'a'|'b' ? never : 'a'  => never
// 'b' extends 'a'|'b' ? never : 'a' => never
// 'c' extends 'a'|'b' ? never : 'a' => 'c'
// 'd' extends 'a'|'b' ? never : 'a' => 'd'
// 最终结果是 将符合条件的类型组合成联合类型 即： 'c' | 'd'

```





## 映射类型

> 映射类型是我们从旧类型中创建一种新的类型，在此之前来了解一下以下知识点

`keyof`索引类型查询,keyof将类型属性联合起来作为一种类型如
```ts
interface I {
  name:string;
  age:number;
}
//下面两种类型写法是等价的
type R = keyof I
type R = "name" | "age"
```

`T[K]`索引访问操作符 对具有索引的类型获取具体索引对应的类型，结合泛型和extends使用

```ts

function getObjectValue<T,K extends keyof T>(obj:T,key:K):T[K]{
return obj[key]
}

const obj = {
  name: '',
  age: 0
}

getObjectValue(obj,'name') // return string type
getObjectValue(obj,'age') // return number type

//注意 K extends keyof T 意味着 K的参数只能是 T的联合类型，即 'name' | 'age'的子集


/**
 * 虽然在泛型中，我们习惯称extends为约束而不是继承，但其实它是属于“继承”的概
 * 念，也就是说， K 实际上是继承自 T的联合类型
 * 就上述代码为例： getObjectValue(obj,'name')
 * keyof T === 'name' | 'age'
 * K === 'name'
 * K 继承自 keyof T 也就是说 'name' 是 'name'| 'age'的子集
 * 在我们认知中 “子集” 应该是类型更多才对，为什么反而 keyof T是父集而不是子集
 * 其实就是联合类型的错觉， 联合类型越多，说明类型越不具体，联合类型越少，越具体，这符合“子集”的概念，因此，在泛型约束中extends也跟继承是同一个概念
 */

```

> 了解了以上知识点之后，我们再来看映射类型，映射类型是从旧的类型中创建新类型，比如如下例子：

```ts
interface I {
  name: string;
  age: number;
  h: boolean
}

type PickType<T, K extends keyof T> = {
  [P in K]:T[P]
} 

let res:PickType<I,'name' | 'age'> = {name:'',age:12}
//res的类型为： {name:string,age:number}


```

```ts
//ExcludeInterface 拓展Exclude ，从某个接口类型剔除另一个接口类型的属性
// Exclude<T,K> 从T中剔除可以赋给K的类型
type ExcludeInterface < T extends K, K > = {
  [ P in Exclude < keyof T, keyof K > ]: T[ P ]
}

interface P {
 name: string,
 age: number
 }

 interface S {
 age:number;
}

let res:ExcludeInterface<P,S> = {name:''}
```

上面例子中，使用了 for in的用法，有点类似索引签名，K继承自`keyof T`,也就是 `'name'|'age' `，那么 P in K 就相当于在遍历这个联合类型 并将索引类型操作符拿到的类型作为属性对应的类型

通过映射类型，我们可以实现更多高级用法，在typescript中，包含了一些内置的映射类型，比如上面我们实现的PickType实际上在ts中有内置类型Pick，当然还包含了许多内置类型，有兴趣可以去看 [高级类型-映射类型](https://www.tslang.cn/docs/handbook/advanced-types.html)

```ts
type NullablePerson = { [P in keyof Person]: Person[P] | null }
type PartialPerson = { [P in keyof Person]?: Person[P] }
```

### 类型推导 infer

> 类型推导是ts在2.8版本以后提供的关键词， 其结合extends语句使用
当我们不确定已有类型定义的类型时，可以用推导类型来动态获取。

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

function getName(id:number):string {
  return ''
}
// typeof getName 的类型为： (id:number) => string
//对应类型推导中的infer R 所以 R是string类型

let res:ReturnType<typeof getName> // string类型


//-----------------------------------------------

type PromiseType<T> = T extends Promise<infer R> ? R : any

const f = new Promise<string>((reslove) => {
  reslove('success')
})

let res:PromiseType<typeof f> //string

```


## 类型保护

>  类型保护就是一些表达式，它们会在运行时检查以确保在某个作用域里的类型。 要定义一个类型保护，我们只要简单地定义一个函数，它的返回值是一个 类型谓词

就像vue3中提供的 isRef、isReactive 等方法，为了确认某个联合类型的变量具体是某种类型，我们需要自定义一个类型保护。


### 自定义类型保护

> 返回值形如 parameter is type 的类型谓词
```ts
class A {
  getName(){}
 }
class B {
  getValue(){}
 }

type AllType =  A | B | null

let res:AllType

function isNull(value:AllType): value is null{ //类型谓词
  return value === null
}

function isA(value:AllType){ //注意此处我们没有写类型谓词
  return  (<A>value).getName !== undefined
}

function test() {
  
if (isNull(res)) {
  console.log('is null')
}
if (isA(res)) {
  console.log('is A')
  res.getName() //error res 可能为null
}
}

```

在isA没有返回类型谓词的时候，在条件语句块里面，res仍然可能是其他类型，而使用了类型谓词后，在语句块里面就已经确保了，res只能为A类型


### typeof 保护类型

> Typescript可以识别 typeof === 'typename' 为保护类型，"typename"必须是 "number"， "string"， "boolean"或 "symbol"。 但是TypeScript并不会阻止你与其它字符串比较，语言不会把那些表达式识别为类型保护

这意味着我们没必要像自定义类型保护一样，写一个函数然后返回类型谓词，让ts识别这是一个类型保护，而是可以直接使用

```ts
let n: string | number
function test() {
  if (typeof n === 'string') { //此处已经被识别为类型保护，那么在if分支就已经确认了n是string类型，具有length属性
    console.log(n.length)
  } else { //else分支不是string属性，那么就只能是联合类型中的number类型
    console.log(n.toString())
  }
}
```

### instanceof类型保护

```ts
class A {
  getName(){}
 }
class B {
  getValue(){}
 }


let res:A | B | null


function test() {
  
  if (res instanceof A) {
    res.getName()
}
}
```


## 类型兼容性

> TypeScript里的类型兼容性是基于结构子类型的。 结构类型是一种只使用其成员来描述类型的方式。

```ts
interface Named {
    name: string;
}

class Person {
    name: string;
}

let p: Named;
// OK, because of structural typing
p = new Person();
```

## 类型兼容性

>类型兼容性用于确定一个类型是否能赋值给其他类型。

### 基本规则


```ts
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: 'Alice', location: 'Seattle' };
x = y;
```
> 结构类型检测 源类型(Y)是否符合目标类型(X)，符合即兼容


```ts
function greet(n: Named) {
    console.log('Hello, ' + n.name);
}
greet(y); // OK
```
> 函数的【参数】，规则也一样


### 两个函数比较

  #### 参数列表比较

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```
> 函数参数列表，允许忽略传参，因此参数数量较少的允许赋值给参数数量较多的类型，前提是参数类型一致

#### 返回值类型比较
```ts
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error, because x() lacks a location property
```
> 类型系统强制源函数的返回值类型必须是目标函数返回值类型的子类型。


> 在TypeScript里，有两种兼容性：子类型和赋值。 它们的不同点在于，赋值扩展了子类型兼容性，增加了一些规则，允许和any来回赋值，以及enum和对应数字值之间的来回赋值。

```ts
let x: number
let y: any
x = y // ok
y = x // ok
```


### 变体

对类型兼容性来说，变体是一个利于理解和重要的概念。

>对一个简单类型 Base 和 Child 来说，如果 Child 是 Base 的子类，Child 的实例能被赋值给 Base 类型的变量

- 协变（Covariant）：只在同一个方向；
- 逆变（Contravariant）：只在相反的方向；
- 双向协变（Bivariant）：包括同一个方向和不同方向；
- 不变（Invariant）：如果类型不完全相同，则它们是不兼容的

## 对比函数的时候需要注意的问题

### 协变（Covariant）：返回类型必须包含足够的数据。

```ts
interface Point2D {
  x: number;
  y: number;
}
interface Point3D {
  x: number;
  y: number;
  z: number;
}

let iMakePoint2D = (): Point2D => ({ x: 0, y: 0 });
let iMakePoint3D = (): Point3D => ({ x: 0, y: 0, z: 0 });

iMakePoint2D = iMakePoint3D;
iMakePoint3D = iMakePoint2D; // ERROR: Point2D 不能赋值给 Point3D
```


### 可选的（预先确定的）和 Rest 参数（任何数量的参数）都是兼容的：

```ts
let foo = (x: number, y: number) => {};
let bar = (x?: number, y?: number) => {};
let bas = (...args: number[]) => {};

foo = bar = bas;
bas = bar = foo;
```


## 拓展

### 可选函数和函数的参数this

```ts
interface ComponentOptions {
  provide?(this: ComponentPublicInstance):object
}
```

> 上述接口的描述中，provide为一个可选的函数，当实现这个接口的时候provide函数可有可无。

参数this，是为函数确定函数内部this的类型，在实际编译后并不会作为参数，只是类型校验。且this参数只能放在首位。如下例子


```ts
interface I {
  init?(x:number,y:number):void
}


class F implements I {
  name = 'FF'
  init(this:F, x: number, y?: number) { //指定this类型是类实例类型F，函数内部则只能使用该this类型的数据
    console.log(this.name,x,y)
  }
}

var f = new F()
f.init(1,2) // 依然是传入1-2个参数，this参数占位忽略
var k = f.init // this指定类型有个好处，就是防止将函数赋值给其他对象的时候，由于this指向改变，导致结果不是自己想要的
k(1,2)// f的init函数赋值给了 k ,this指向了其他地方，可能是window或void，与参数指定的F类型不一致，则提示错误。
```

> 所以在写类的公共方法的时候，出于严谨考虑，可以使用this参数去代替 函数绑定this。
这样就无需再构造函数中写`this.xxx.bind(this)`，直接在编码过程中发现问题


## vue组件ref引用类型

> 当我们在组件中设置ref属性的时候，我们可以获得组件的实例。为了能正确识别ref引用类型，我们应该正确设置类型

```ts
<Component ref="refComp"></Component>


import Component from './Component'

const refComp = ref() //同名的ref，则拿到了Component实例，但是此时的refComp为any类型，为了正确识别我们应该使用下面类型处理

const refComp = ref<InstanceType<typeof Component>>() //InstanceType能获取构造函数的实例类型， 而vue3组件导出，都使用了DefineComponent函数，其返回类型为 构造函数本身，所以typeof操作符拿到Component的类型再获取其实例类型，就能正确识别这个组件引用类型
```