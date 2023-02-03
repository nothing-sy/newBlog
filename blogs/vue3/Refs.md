---
title: Refs
date: 2022-2-18
categories:
 - vue3
---

# Refs

Refs的内容包括ref对象及相关的ref处理函数，在vue3里面，为了统一数据行为，普通的数据类型也要求封装成引用类型的数据，以达到和对象一致的行为表现。

但ref和reactive是有区别的。

- ref的入参可以是任意类型，如果是object类型，则调用reactive处理。 其余情况则为该值创建一个RefImpl的实例， 该实例由class RefImpl生成，包含了一个私有属性value,以及get /set方法
- reactive 只接受一个 类型为 object的参数，其余类型会提示错误， 然后通过get方法对用到的属性做代理（proxy），proxy里面包含了set/get方法



1. ref

接受一个内部值并返回一个响应式且可变的 ref 对象，ref 对象仅有一个 `.value` property，指向该内部值。

【如果将对象分配为 ref 值，则它将被 [reactive](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 函数处理为深层的响应式对象】

```js
const obj = ref({a:1}) //传入object将会通过reactive生成一个代理，并赋值给value属性
console.log(obj) //RefImpl类型
console.log(obj.value) // proxy类型 => 也就是reactive(obj)的返回值

```



2. unref

如果参数是一个 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref)，则返回内部值，否则返回参数本身。这是 `val = isRef(val) ? val.value : val` 的语法糖函数

3. toRef

可以用来为源响应式对象上的某个 property 新创建一个 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref)。然后，ref 可以被传递，它会保持对其源 property 的响应式连接。

```js
function toRef(object, key, defaultValue) {
    const val = object[key];
    return isRef(val)
        ? val
        : new ObjectRefImpl(object, key, defaultValue);
}
```

```text
toRef接受一个object，如果里面的属性是ref对象，则直接返回，否则将属性对应的值转成ObjectRefImpl对象，所以最终toRef会返回一个ref或者ObjectRefImpl对象更改这个objectRefImpl对象会影响原始数据
```





4. toRefs

将响应式对象转换为普通对象，其中结果对象的每个 property 都是指向原始对象相应 property 的 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref)



> ```js
> function toRefs(object) {
>     if (!isProxy(object)) {
>         console.warn(`toRefs() expects a reactive object but received a plain one.`);
>     }
>     const ret = shared.isArray(object) ? new Array(object.length) : {};
>     for (const key in object) {
>         ret[key] = toRef(object, key);
>     }
>     return ret;
> }
> ```
>
> 
>
> ```text
> toRefs只接受一个proxy对象，然后对该对象的所有值通过toRefAPI处理
> ```





5. isRef

检查值是否为一个 ref 对象

6. customRef

创建一个自定义的 ref，并对其依赖项跟踪和更新触发进行显式控制。它需要一个工厂函数，该函数接收 `track` 和 `trigger` 函数作为参数，并且应该返回一个带有 `get` 和 `set` 的对象

>自定义Ref 相当于 替换了RefImpl类，并自定义set和get以达到想要的额外目的，比如在设置这个自定义ref的值的时候，打印一下值。
>
>为了保证vue本身的响应和依赖收集， 这个工厂函数接收了track和trigger分别对应 依赖跟踪 和 更新触发机制，保持和RefImpl一致性(以下代码为RefImpl类，其trackRefValue，triggerRefValue对应了自定义ref的track和trigger)
>
>```js
>class RefImpl {
>    constructor(value, __v_isShallow) {
>        this.__v_isShallow = __v_isShallow;
>        this.dep = undefined;
>        this.__v_isRef = true;
>        this._rawValue = __v_isShallow ? value : toRaw(value);
>        this._value = __v_isShallow ? value : toReactive(value);
>    }
>    get value() {
>        trackRefValue(this);
>        return this._value;
>    }
>    set value(newVal) {
>        newVal = this.__v_isShallow ? newVal : toRaw(newVal);
>        if (shared.hasChanged(newVal, this._rawValue)) {
>            this._rawValue = newVal;
>            this._value = this.__v_isShallow ? newVal : toReactive(newVal);
>            triggerRefValue(this, newVal);
>        }
>    }
>}
>```



### Demo:

```html
<p>
{{a}}
</p>
<el-button type="primary" @click="change">改变</el-button>
```



```js
const createCustomRef = (value:number) => {
  return customRef((track,trigger) => {
    return {
      get(){
        track()//跟踪依赖
        return value
      },
      set(v:number) {
        value = v
        trigger() //触发更新
      }
    }
  })
}
const a = createCustomRef(0)
const change = () => {
  a.value++
  }
```



7. shallowRef

创建一个跟踪自身 `.value` 变化的 ref，但不会使其值也变成响应式的。这是一个很有用的API。 当不需要关注值内部的变化时可以用这个，比如，图表的配置。我只关心最后配置的结果，而不关心配置的对象内部值的变更。

前面介绍响应性基础API的时候讲到关于shallowReactive或者markRaw来处理数据，只对浅层次的属性转换成proxy。而shallowRef更符合这个场景。如下

```js
const options = shallowRef({})
options.value = {
    tooltip: {}
    //....
                }
//当value变更时，才会触发更新， 而数据内部属性变更，并不会触发更新，比如。 options.value.a=1并不触发 ，如果这个options是由ref创建，则深层的数据也会响应。shallowRef的本质是，传入一个object对象，不使用reactive处理数据，所以options.value指向的只是 入参的地址，不会被代理



```



8. triggerRef

手动执行与 [`shallowRef`](https://v3.cn.vuejs.org/api/refs-api.html#shallowref) 关联的任何作用 (effect)。简单来讲就是，手动触发深层次的数据更新

```js
const shallow = shallowRef({
  greet: 'Hello, world'
})

// 第一次运行时记录一次 "Hello, world"
watchEffect(() => {
  console.log(shallow.value.greet)
})

// 这不会触发作用 (effect)，因为 ref 是浅层的
shallow.value.greet = 'Hello, universe'

// 记录 "Hello, universe"
triggerRef(shallow)
```

