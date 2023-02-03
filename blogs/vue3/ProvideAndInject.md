---
title: Provide and inject
date: 2022-2-22
categories:
 - vue3
---

# Provide和inject

provide和inject的作用在于，当组件层级嵌套较深的时候，替代props层层传递，获取数据的方法。

也可以在兄弟组件需要用到同样数据的场景下，把数据往上提一层， 而不是使用vuex

用法比较简单

```js
//祖先组件

import { provide } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    provide('location', 'North Pole') //在setup函数内，直接调用provide函数，传入key和value,在深层次组件中调用inject读取key
    provide('geolocation', {
      longitude: 90,
      latitude: 135
    })
  }
}

```

```js
//孙组件
import { inject } from 'vue'

export default {
  setup() {
    const userLocation = inject('location', 'The Universe') //通过inject读取祖先组件中provide的值。 
    const userGeolocation = inject('geolocation')

    return {
      userLocation,
      userGeolocation
    }
  }
}
```

以上两块代码就是最基本的依赖注入。 如果要保持数据响应性，即provide的数据变更inject注入的数据也变更，则provide传入的值应该为响应式数据，如下：

```js
const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    provide('location', location)
    provide('geolocation', geolocation)
```

> 当使用响应式 provide / inject 值时，**建议尽可能将对响应式 property 的所有修改限制在\*定义 provide 的组件\*内部**
>
> 也就是不要在孙组件里面去更改这个注入的值。而始终在提供provide的组件内部更改。如果涉及到需要在inject组件更改值，则可以通过调用祖先组件provide的方法。例子如下

```js
    const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    const updateLocation = (locationName) => {
      location.value = locationName
    }

    provide('location', readonly(location)) //为了防止子组件更改了这个值，对响应式数据转成readonly
    provide('geolocation', readonly(geolocation))
    provide('updateLocation', updateLocation) //提供一个方法供注入的组件调用
```



