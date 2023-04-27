---
title: Computed,Watch and Props
date: 2022-5-26
categories:
 - vue3
---

# Computed and Props
> vue3的响应API很多，ref,reactive,readonly,shallow....等等，导致数据的响应类型也增多，而vue2只要知道这是一个响应的数据即可，不管数据是引用类型还是基本数据类型，都能以统一的方式处理，比如watch和computed。  因此，在学习vue3过程中导致对数据类型掌握不准确，无法清晰知道数据能否响应或被监听

### props
关于传入的属性，通过defineProps宏函数处理过后，会返回一个proxy对象，结合之前学习的reactive处理数据成proxy对象，就会【误以为】，被代理后的数据，只要属性是一个对象，也会同样被处理成proxy这样一个误区

在经过一系列的观察以后，发现defineProps虽然把props处理成了proxy但是并不是使用了类似于reactive的方式，在get捕获器中也将值为对象的对象转变成proxy.

以下是vue3的源码中的一段
```js
  /**
   * Used to create a proxy for the rest element when destructuring props with
   * defineProps().
   * @internal
   */
  function createPropsRestProxy(props, excludedKeys) {
      const ret = {};
      for (const key in props) {
          if (!excludedKeys.includes(key)) {
              Object.defineProperty(ret, key, {
                  enumerable: true,
                  get: () => props[key]
              });
          }
      }
      return ret;
  }
```
:::warning 结论！

上述代码中很明确的表示，当使用defineProps()时会使用这个方法处理。 它将返回一个proxy，但是该proxy的get方法只是单纯的返回了props[key]，
这意味着，props传入的时候本身是什么类型，那就返回什么类型。并不会像reactive一样做深度处理，把值为对象的属性继续转换成proxy对象

:::



```js
//父组件
const num = ref(1)
<son :num="num">

//子组件
const props = defineProps({num:{type:Number}})
//此时props为一个proxy，而num属性 为 父组件的 num变量，
//是一个ref。因此，父组件num在变更值的时候，props.num也会相应变更

```


```js
//父组件
const obj = reactive({a:{b:1}})
<son :obj="obj">

//子组件
const props = defineProps({obj:{type:Object}})

//此前我对definedProps有一个【误解】，因为被宏编译以后拿到的props变量
//总是proxy对象，那么props.obj里面的对象也会被自然处理成proxy。
//（reactive函数的处理方式，在get函数里面总会将object数据再次用
//reactive函数处理），实际上并非如此，正如上面所言，props的属性只是一个引用
//我们来证实一下

//如果父组件的obj不是一个reactive函数处理过的数据，而是普通的对象
//在控制台输出props.obj.a你会发现是个普通的对象{b:1}而不是proxy
//因此在computed和watch函数中也无法触发副作用。

//如果父组件中obj是一个shallowReactive({a:{b:1}})
// obj.a={b:2} 会响应
// obj.a.b=3  不会响应，因为使用的是shallow只对第一层数据变化有响应

由此可见，defineProps定义的是一个引用。
返回的对象虽然是个proxy对象，但是内部的值是否响应跟父级变量有关，
而不会因为props.xxx是一个对象主动处理成proxy. 
理解了这个让我们使用watch和computed函数中能主观意识到，副作用函数能否被触发

```

### watch 和 computed 与props

上面已经简述了关于defineProps宏的作用，那么我们就可以更好的理解watch和computed

watch 侦听器数据源可以是返回值的 getter 函数，也可以直接是 ref

```js

const num = ref(1)
const state = reactive({ count: 0 })
//可以被检测到
watch(
  num,
  (count, prevCount) => {
  }
)
//可以被检测到
watch(
  () => state.count, //state.count是基本类型，所以得以getter函数方式返回。当然如果state是个普通对象而不是proxy，也是无法监听的
  (count, prevCount) => {
  }
)
//可以被检测到
watch(
  state,
  (count, prevCount) => {
  }
)

//假设obj = reactive({a:{b:1}})
const props = defineProps({obj:Object})

//下面可以被检测到
watch(
  props, //props本身是proxy，所以下面的属性变更都会被监听到
  (count, prevCount) => {
  }
)


//vue会告警，因为是基本类型，需要用getter ()=> props.obj.a.b
watch(
  props.obj.a.b, 
  (count, prevCount) => {
  }
)

//假设obj = shallowReactive({a:{b:1}})
//下面会告警，因为shallowReactive创建 obj.a不会被处理成proxy，所以需要用getter函数返回
obj.a = {b:2}
watch(
  props.obj.a, 
  (count, prevCount) => {
  }
)


//下面不会被检测到，与上同理
obj.a = {b:2} //下面即使改成getter  () => props.obj.a.b也不会触发副作用，
//因为props.obj.a.b本身不具备响应性，而上面的例子中 a属性具备响应性，所以
//getter函数() => props.obj.a 能触发副作用
watch(
  props.obj.a.b, 
  (count, prevCount) => {
  }
)


```

### computed 与watch的机制特性一致，对于什么类型的数据能够响应，取决于数据类型，凡是能被watch 监听到的数据，在computed中都能响应
