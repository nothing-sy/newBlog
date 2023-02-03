---
title: 组合式API
date: 2022-1-27
categories:
 - vue3
---

# 组合式API
> 组合式API是vue3的一大特性，vue3为了兼容vue2的特性，仍然保留了vue2的写法。其中vue3的一大特点就是组合式API。

人总是习惯性地待在舒适区，在真正使用vue3写法之前，我一直觉得所谓的组合式API完全没必要。写法复杂，步骤繁琐，需要每次引用好多方法处理数据，监听，计算属性等（这个问题其实也可以通过插件解决，无需每次引入。后面章节会讲到）。 先来谈谈我个人对组合式API的看法。

所谓组合式 API就是一组低侵入式的、函数式的 API，使得我们能够更灵活地「组合」组件的逻辑，而不需要使用option选项来处理逻辑代码。在量级较少的组件中，组合式API可能无法体现其优势，但是当业务功能越来越多，option的配置只会越来越长。相同功能的逻辑代码就会东一块西一块。不好处理。这时候我们肯定会想到，这问题好解决啊，我用vue2写法，然后使用mixin,或者把 功能模块的方法写到不同的文件，再引入不就行了吗？

我们来探讨下这两种写法是否能替代组合式API的优点。

- mixin
用混入的方式，我们可以把功能模块相关的属性，数据，方法单独抽离出来， 这个看似解决了  【用组合式API解决 逻辑代码分散，关注点分散】的问题，
首先，如果我们使用多个mixin，我们就很难在入口组件里面，区分哪个方法，哪个变量是来自于哪个mixin。
其次，我们可能遇到过多个相似的功能，引入同一个mixin的场景，比如A,B两个组件都有init方法，init方法会调用A独有的属性，此时B组件没有这个属性，我们就会写一些多余的判断条件去处理，比如先保证这个变量是否存在，再处理。 这里就增加了业务的耦合度。（相信我，遇到这种情况，一般人不会选择把相同的mixin部分抽出来，再去拓展不相同的部分，然后分成两个不同的minxin引入），所以mixin并不能很好地完成 逻辑代码分散并且使得代码更易读。
- 区分功能模块，分离数据、方法等
这种方式只能简单的分离非响应式数据相关的部分，比如，我们把data选项里面的数据抽离出去

```js
//data.js
  export const data1 = {
  a:1
  }
  
//methods.js
 export const method1 = () => {}

//main.vue
import {data1} from data.js
import {method1} from methods.js
//组件内
data () {
return {
...data1
}
},
methods: {
method1
}
```



从上面看来，我们很容易看出我们确实可以一定程度的分离出不同功能模块的内容，但是我们始终还是要在options里面将所有的响应式数据，计算属性等糅合在一起。但缺点也很明显，我们无法清晰知道data1里面是什么数据（除非数据一 一导出）， 如果我要在方法method1里面更改响应式数据，只能把this对象传进去才能更改。因为我们更改响应式数据是```this.xxx = xxx```这种写法, this指向了组件实例。很明显这是不切实际的。所以 这种区分功能模块，分离部分数据、方法的方式，并不能很好的完成业务代码分离的问题。



我们再来看看vue3的组合式API。 vue3将响应式数据，监听器，计算属性等封装成一个函数。 让其能在任何地方单独处理，组合。而不局限于 选项。

比如，响应数据通过 ref、reactive、toRefs、toRefs等函数将数据处理成统一的响应式数据

```js
import {ref,watch,computed} from 'vue' //这里只简单介绍ref,其余的可去官网了解使用方式
const count = ref(0) // ref接收一个值并返回ref对象，该对象可响应，
count.value++ // 经过ref包装的数据都被统一处理成 {value:xx}的形式，真正操作的就是 .value属性， 而在模板中，该对象会被自动解析，无需使用.value就可以读取值，直接写count即可


//watch 侦听器数据源可以是一个具有返回值的 getter 函数，也可以直接是一个 ref 。watch相关的API有好几个可自行了解
watch(count,() => {
    //do something
    //当count数据变更时就会触发
})

const summary = computed(() => {return `计数器：${count.value}`})
//summary.value拿到的永远都是最新的count.value对应的数据
```

简单的介绍了最主要的三种api之后，我们很明显能发现，响应数据，监听器，计算属性都不在局限于vue组件的选项中。 他们可以在单独的js文件中处理。（而且响应式数据也可以随意导出，其响应特性一直存在）

我们来写一个简单的例子，一个 用户信息看板功能，其子功能包括： 1、搜索客户栏  2 客户信息列表 （我们尽可能拆分细一些，用来描述组合API的优点）

// cusList.js  搜索客户栏

```js
import {ref} from 'vue'
export const cusList = ref([]) //搜索到的客户列表
//通过客户名模糊搜索客户列表
export const getCustomerList = async (name) => {
    //这里是你的异步方法，拿到客户数据列表
    //此处假设已经拿到了客户id数据
    cusList.value = [
            {name: '客户1',id:1},
            {name: '客户2',id:2},
        ]
}

```



//  cusDetail.js  客户详细信息数据

```js
import {ref} from 'vue'
export const  cusDetail = ref({})//搜索到的客户信息
//根据客户ID，查询客户的详细信息
export const getCusDetail = (id) => {
    //假设已经拿到了详细信息
    const res = {
        name: '客户' + id,
        adress:'客户地址',
        age:'年龄',
        mobile:'xxx'
    }
    cusDetail.value = res 
}


```



// CusBoard.vue 用户信息看板组件

```vue
<template>
<div>
<el-select
    v-model="customerId"
    filterable
    remote
    reserve-keyword
    placeholder="Please enter a keyword"
    :remote-method="getCustomerList"
    @change="getCusDetail"
  >
    <el-option
      v-for="item in cusList"
      :key="item.id"
      :label="item.name"
      :value="item.id"
    >
    </el-option>
  </el-select>
    <div>
    {{cusDetail}}
    </div>
 </div>
</template>

<script setup> //setup是语法糖，相当于导出组件， 而且引入子组件直接import 即可直接使用，无需再在components选项里面注册
import {ref} from 'vue'
import {getCusDetail,cusDetail} from './components/cusDetail'
import {cusList,getCustomerList} from './components/cusList'
const customerId = ref('') //查询的客户ID

//程序到此结束，很明显，该vue文件很简洁，cusDetail和cusList导出的响应式数据可以直接在模板里面使用。
//我们能清晰知道cusDetail是负责什么功能,cusList是负责什么功能

</script>
```



由上面三段代码实现的一个 查询客户信息功能可以看出。 数据不再被选项options限制。可以在任意被分解的模块使用，并导出。

我们可以利用这个特性，将独立的功能，拆分出来。使得代码足够清晰。
