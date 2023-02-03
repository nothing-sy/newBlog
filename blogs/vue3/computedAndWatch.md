---
title: Computed and Watch
date: 2022-2-21
categories:
 - vue3
---

# Computed and Watch

1. computed

接受一个 getter 函数，并根据 getter 的返回值返回一个不可变的响应式 [ref](https://v3.cn.vuejs.org/api/refs-api.html#ref) 对象。 

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // 错误,因为该计算属性没有设置set方法，默认的set方法会提示错误
```

**注意！computed的getter函数，返回的值必须为可响应的，即返回值包含ref或者proxy的表达式，否则计算属性不会随着数据更新而更新，如下代码**

```js
let obj = shallowReactive({a:{b:1}})
const c = computed(() => obj.a.b) //当obj.a.b数据变更时c.value不会变更，因为obj是shallow类型，深层次属性不是proxy类型，也就是不具备响应性，因此computed也不会触发回调函数，返回最新值
```

【补充】：对computed与传入属性props的直接的联系，请查看文章 - [Computed,Watch and Props] (https://heartofblack.github.io/Blog/blogs/vue3/computedAndProps.html)

2. watchEffect

立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。等同于vue2中watch 响应数据中的option为{immediate:true}， 但是优势在于不必要显性传入一个 被侦听的数据源。 只要副作用函数里面的依赖更新，则会立即触发回调。 当多个依赖被同时更新时，只会触发一次。

```js
const obj = reactive({a:{b:1}})
const n = ref(1)
watchEffect(() => {
  console.log('触发监听', n.value + obj.a.b)
return n.value + obj.a.b
})
//无论是单独变更n还是obj.a.b的值，watchEffect的副作用函数都会被触发。 如果N个依赖被连续更新，函数只会被触发一次
n.value++
obj.a.b++

```



类型声明如下：

```js
function watchEffect(
  effect: (onInvalidate: InvalidateCbRegistrator) => void,
  options?: WatchEffectOptions
): StopHandle  //返回一个stopHandle函数， 可调用这个函数，停止监听

interface WatchEffectOptions {
  flush?: 'pre' | 'post' | 'sync' // 默认：'pre' //改参数指定回调什么时候触发， pre为模板更新前调用， post为模板更新后调用  sync为同步调用
  onTrack?: (event: DebuggerEvent) => void
  onTrigger?: (event: DebuggerEvent) => void
}
  
```

 `watchPostEffect`、`watchSyncEffect`为watchEffect中 options配置flush参数的别名。



3. watch

`watch` API 与选项式 API [this.$watch](https://v3.cn.vuejs.org/api/instance-methods.html#watch) (以及相应的 [watch](https://v3.cn.vuejs.org/api/options-data.html#watch) 选项) 完全等效。`watch` 需要侦听特定的数据源，并在单独的回调函数中执行副作用。默认情况下，它也是惰性的——即回调仅在侦听源发生变化时被调用。

**注意！watch接受的侦听源为响应式数据，即可以是ref或者getter函数(该getter函数返回值可以为普通数据类型，而不是响应数据)或者proxy。 要注意的是reactiveAPI处理的object数据当属性值不为object时不会被处理成proxy对象而是返回原始类型，因此不具备响应性，不能被watch侦听**

demo:

```js
const obj = reactive({a:{b:1}})
watch(obj.a.b, value => {
    // obj.a.b是number类型，不会触发副作用函数。
})


watch(() => obj.a.b, value => {// 虽然obj.a.b是number类型，但是此处作为getter函数，可以触发副作用。当然前提是obj是个响应数据而不是原始数据
console.log('这种写法可以触发')
    
})
```

以上代码中obj.a.b是number类型，不会触发副作用函数。




>- 与 [watchEffect](https://v3.cn.vuejs.org/api/computed-watch-api.html#watcheffect) 相比，`watch` 允许我们：
>  - 惰性地执行副作用；（当监听的ref或者getter函数变化时，才会执行副作用）
>  - 更具体地说明应触发侦听器重新运行的状态；（可以显示地设置flush/immediate/deep参数）
>  - 访问被侦听状态的先前值和当前值。

watch侦听多个数据源

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```



### 关于watch和watchEffect相似之处

`watch` 与 [`watchEffect`](https://v3.cn.vuejs.org/api/computed-watch-api.html#watcheffect) 在[手动停止侦听](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#停止侦听)、[清除副作用](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#清除副作用) (将 `onInvalidate` 作为第三个参数传递给回调)、[刷新时机](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#副作用刷新时机)和[调试](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#侦听器调试)方面有相同的行为。

>重点讲解 - 清除副作用
>
>- 副作用即将重新执行时
>- 侦听器被停止 (如果在 `setup()` 或生命周期钩子函数中使用了 `watchEffect`，则在组件卸载时)
>
>清除副作用的目的是为了做一些默认操作， 当被监听对象或者跟踪的依赖数据变更时，希望把上一次的副作用清除掉，就可以在这个函数中处理。
>
>比如，当更新客户ID变更时，把上一次的客户内容清空，这时候就可以把清空函数放在这个清楚副作用回调中处理。大多数时候，副作用都是异步请求数据。如果数据还未请求完成， 依赖又变更了，那么我们应该重新请求最新数据，上一次的请求应该被终止， 则清除副作用回调中，应该写终止请求

