---
title: 响应性基础API
date: 2022-2-16
categories:
 - vue3
---

# 响应性基础API

> 响应性基础API，是一些涉及响应式数据相关的函数。其基本实现原理是用proxy来代替vue2的Object.defindProperty方法拦截属性的设置，以下代码为简单模拟proxy处理数据

```typescript
const createReactive = (target:any):any => {
  return new Proxy(target,{
    set(target,key:any,value){
      return Reflect.set(target,key,value)
    },
    get(target,key:any) {
      const res = Reflect.get(target,key)
     if (typeof res === 'object') { //如果是object继续调用reactive方法
         //写在get方法，而不是在set方法里面递归代理就是vue3的优点之一，只有在触发get方法，也就是代码中需要访问到某个属性的时候，才会触发get，如果是object则调用reactive方法返回代理，否则返回原始数据
       return reactive(res)
      }
        return res
    }
  })
}

const reactive = (target:any) => {
return createReactive(target)
}

const ps = reactive({value:{}})

 ps.value = {a:{b:{c:{d:{e:10}}}}}
 console.log(ps,ps.value,ps.value.a,ps.value.a.b) //只有当这些属性被访问到时，这些深层次属性才会被代理，否则不会操作。 当这些属性被代理之后。赋值触发set方法，进而做更新数据的操作，这个例子只简单介绍proxy，不涉及其他部分

```





1. reactive

reactvie接受一个object原始数据，并返回响应式代理，其响应是“深层次”的

操作响应式代理或者object都能改变object的值。但是如果要使其触发响应，必须操作返回的响应式代理。

```js
const p = reactive({count:0})
p.count++ // 值为1
```

vue3实现响应式是通过proxy处理的，所以vue2中存在的弊端，如对象新增属性，数组下标赋值等操作已经解决，不再需要通过特殊的vue.$set方法赋值。 vue3中直接新增属性，数组下标赋值均会通过proxy的拦截器处理。

```js
//reactive会自动解包对象属性下面的所有refs，且保持这些ref的响应特性

const count = ref(1)

const obj = reactive({ count }) 

 //以上代码会自动解包ref类型count,不再需要通过count.value读取值，而是直接obj.count读取，且保持数据响应
```



2. readonly

接受一个对象 (响应式proxy对象或纯对象) 或 [ref](https://v3.cn.vuejs.org/api/refs-api.html#ref) 并返回原始对象的只读代理。只读代理是【深层的】：任何被访问的嵌套 property 也是只读的

```js
const original = reactive({ count: 0 })
const copy = readonly(original)

original.count++ //正常
// 变更副本将失败并导致警告（其只读特性是深层次的，如果count为对象，其属性也是只读）
copy.count++ // 警告!
```



3. isProxy

检查对象是否是由 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 或 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 创建的 proxy



4. isReactive

检查对象是否是由 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 创建的响应式代理



5. isReadonly

检查对象是否是由 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 创建的只读代理



6. toRaw

> 使用场景： 当一个复杂数据存在多个被代理对象时，可以通过这个方法拿到原始数据用于传参
>
> 该方法处理 内部属性被reactive/readonly/ref等方式处理的原始数据
>
> 比如：
>
> ```js
> const customerId = ref('')
> const obj = reactive({
>     shallowProp: { 
>         deepProp: { 
>             
>         }
>     },
>     customerId,
>     r: reactive({}), 
>     shallowProp1: {} 
> })
> console.log(toRaw(obj))//customerId属性仍然是ref对象，r属性仍然是proxy对象
> ```
>
> 

返回 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 或 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 代理的原始对象。这是一个“逃生舱”，可用于临时读取数据而无需承担代理访问/跟踪的开销，也可用于写入数据而避免触发更改。**不**建议保留对原始对象的持久引用。请谨慎使用



7. markRaw（常用）

标记一个对象，使其永远不会转换为 proxy。返回对象本身。这个使用场景会经常见到，当一些深层次数据，而我们又不需要监听这些属性变化的时候，我们可以标记为一个原始数据，使其无法被转换成proxy。比如 业务中配置图表option，而option里面数据的变动我们并不关心。因此我们可以这样写。

```html
<chart :option="options.value"></chart>
```



```js
const options = reactive({value: {}}) //为了监听图表中的变化，我们单独创建了一个value属性来监听数据的变化，我们并不关注value对象内部属性的变化,因此可以把配置好的图表对象用markRaw处理
const baseOption = {
    tooltip:{},
    series: []
    //...
}
options.value = markRaw(baseOption) //这样标记以后，即使后面我们访问了baseOption里面的属性也不会使得该属性转换为proxy对象

```



8. shallowReactive（shallow是浅的意思）

创建一个响应式代理，它跟踪其自身 property 的响应性，但不执行嵌套对象的深层响应式转换 (暴露原始值)【其也适合第7点中提到的图表例子，我们只关心option.value的变化也就是浅层次的数据响应，不关心深层次的数据变化，因此shallowReactive也适用于这个例子】

```js
const obj = shallowReactive({
    shallowProp: { //浅层次属性，其实也就是首层属性代理
        deepProp: { //深层次属性
            
        }
    },
    shallowProp1: {} //浅层次
})

const change = () => {
obj.shallowProp1 = 1 //有效
}
const change1 = () => {
obj.shallowProp.deepProp=3 //无效
}

```

> !与 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 不同，任何使用 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref) 的 property 都**不会**被代理自动解包
>
> 也就是说，当shallowReactive里面传入的对象中存在 ref变量，它不会自动被解包
>
> ```js
> const customerId = ref('1') //查询的客户ID
> const obj = shallowReactive({
>     customerId
> })
> //访问方式为： obj.customerId.value而不是obj.customerId
> ```
>
> 



9. shallowReadonly

创建一个 proxy，使其自身的 property 为只读，但不执行嵌套对象的深度只读转换 (暴露原始值)，跟shallowReactive同理。

> 与 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 不同，任何使用 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref) 的 property 都**不会**被代理自动解包。

