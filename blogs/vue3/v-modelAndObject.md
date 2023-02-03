---
title: v-model和绑定值为object的疑问
date: 2022-5-12
categories:
 - vue3
---

### 疑问来源
>二次封装饿了么组件的时候，我希望传入的form Object prop不会被直接改变，而是由emit去触发数据的变更。当我尝试以`const form = reactive({name:''})` 处理form数据并绑定到组件上。遇到了一些疑问，下面会列举出来

1. 定义数据的时候一般情况下使用const，因为正常情况我们需要变更的是object内部的数据，而并非object本身，因此当我们在内部用update:modelValue去更新form数据的时候就出现问题，首先update:modelValue做了这种操作： `form = $event.value.target`直接将我们的结果赋值给了form而并非更新form里面的值。也就是说form不再是原来定义时候的proxy而是全新的内容，其次因为定义为const，所以也是没法变更数据的。如果你想尝试改成let，你只是解决了其中一个问题，proxy被替换的问题还没有得到解决。

2. 某些eslint规则会默认帮你把let纠正成const（如果你没有显式地变更该变量的话）。因此我们可以考虑把form数据定义成ref或者shallowRef，这样的话我们在update操作的时候始终变更的都是form.value，也能正确地触发数据更新

### 结论
在vue issue上暂未看到相关的bug，但是在其余的网站上也有看到相同情况，表示v-model不能绑定reactive数据。综合实际情况建议以下几种做法：

1. 显式地重写prop和event，但是这种方法已经脱离了v-model这个语法糖了，如下代码
```
<comp :modelValue="obj" @update:modelValue="updateFun"/>


const obj = reactive({name: 'xx'})
const updateFunc = (val) => { //假设val 是{name: 'yy'}
  Objcet.assign(obj,val)
}

```

2.使用ref，如下代码

```
<comp v-model="obj"/>


const obj = ref({name: 'xx'}) //注意ref处理obj会使用reactive处理，即 obj.value 是一个reactive处理过的proxy对象

//这样，子组件emit触发update:modelValue的时候，能够正确更新obj的数据，只是obj.value的数据类型原来是proxy，更新后是什么类型，由返回值决定
```


3.使用shallowRef,如下代码
```
<comp v-model="obj"/>


const obj = shallowRef({name: 'xx'}) //obj.value是个普通object对象

```

上面三种方法都有弊端。
第一种相当于直接舍弃了v-model
后面两者，因为使用的是ref，所以如果父组件需要监听绑定数据的变化，只能监听obj.value变化， 深层数据的变化，不安全、不可靠，因为obj.value数据类型无法保证。

在一般情况下，建议使用最后两者