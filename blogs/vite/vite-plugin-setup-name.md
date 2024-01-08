---
title: 处理script setup重定义组件名

date: 2024-01-08

categories:
 - vite
---

:::场景前提
在vue3.3+版本之前，vue会自动根据文件名生成组件名。
但在实际开发过程中，会有许多index.vue文件，这时候需要我们重定义组件名。
:::

## 方法一

在script标签中添加`name`属性

```vue
<script>
export default {
  name: '组件名'
}
</script>

<script setup>
// do something
</script>

```

## 方法二

使用了插件 `vite-plugin-vue-setup-extend`
该插件原理是，在vite的钩子`transform`中，通过`@vue/compiler-sfc`模块解析到script setup标签，并把标签中的name拿出来，拼接到源码上。拼接的内容如方法一。
这样就完成了简化，只需要在setup中写name属性即可覆盖默认的组件名称


:::优化插件契机
在vue3.3+之前，@vue/compiler-sfc模块是无法解析外部文件引入类型的，比如： type.ts 导出了 `export type Props`
无法在`index.vue`直接使用`defineProps<Props>()`，会提示找不到资源
:::

为了解决这个类型需要写两次的问题，将vue版本升到了3.3+，即可以使用外部引入类型和`defineOptions({name:'xxx'})`来定义组件名。
但是该版本和`vite-plugin-vue-setup-extend`不兼容。`vite-plugin-vue-setup-extend`是调用了`@vue/compiler-sfc`来解析的，经过排查这样调用会警告不处于node环境，无法调用fs模块（具体原因没找出来）。

所以为了解决这个问题，并兼容setup name的写法，特意重写了`vite-plugin-vue-setup-extend`，去掉其使用`@vue/compiler-sfc`模块的内容。代码如下：

```ts
import MagicString from 'magic-string'

/**
 * vite-plugin
 * 兼容原有script setup插件的name写法与vue3.3以上可以从外部引入defineProps的类型声明
 * 如果项目使用vue3.3以上，请保持良好习惯使用defineOptions去定义组件名，而不是使用setup插件的name写法
 */
export default function () {
  return {
    name: 'vueSetupName',
    enforce: 'pre',
    async transform (code:string, id:string) {
      if (!/\.vue$/.test(id)) {
        return null
      }
      const scriptSetupTag = /<script.+setup.+/.exec(code)?.[0]
      // 没找到script setup标签
      if (!scriptSetupTag) return null
      // 单独匹配，防止书写顺序问题导致匹配失败
      const lang = /.+lang=('|")(?<lang>\S+)('|")/.exec(scriptSetupTag)?.groups?.lang
      const name = /.+name=('|")(?<name>\S+)('|")/.exec(scriptSetupTag)?.groups?.name
      const codeStr = new MagicString(code)
      if (name) {
        codeStr.appendLeft(
          0,
          `<script ${lang ? `lang="${lang}"` : ''}>
  export default {
    name: "${name}"
  }
  </script>
  `)
      }
      return {
        map: codeStr.generateMap(),
        code: codeStr.toString()
      }
    }
  }
}

```


### 实现步骤：
1. `magic-string` 可以为字符串源码提供一些方法，例如插入、删除、替换字符串等操作。
2. 判断id是否为vue文件，如果是往下走
3. 使用正则表达式匹配script setup标签，获取其中的lang和name属性
4. 按前面的方法1加上字符串，注意两个script的lang必须保持一致，
5. 返回修改后的源码，因为源码进行了修改，所以最好根据最新的代码生成sourceMap，否则无法正确对应上源码