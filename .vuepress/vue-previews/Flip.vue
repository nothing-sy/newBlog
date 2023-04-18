<script setup>
import {ref} from 'vue'
import {ElButton} from 'element-plus'
const cloneArray = arr => JSON.parse(JSON.stringify(arr))
const defaultList = Array.from({length: 5}, (item,index) => ({
  name:index,
  y: index  * 50 + index * 20,
}))

const list = ref(defaultList)


const revert = async () => {
  const reverList = cloneArray(defaultList).reverse()
  reverList.forEach((item,index) => {
    item.endY = index  * 50 + index * 20
    item.translateY = item.y - item.endY
    item.transform = true
  })
  list.value = reverList
  
 setTimeout(() => {
  list.value = reverList.map(item => {
    return {
      ...item,
      transform: false,
      transition: true
    }
  })
 }, 1e3)
 
  
}

</script>

<template>
  <div class="flex">
  <div class="item" v-for="item in list" :key="item.name"
  :style="{
    transform: `${item.transform ? `translateY(${item.translateY}px)` : 'translateY(0px)'}`
    ,transition: `${item.transition ? 'transform linear 1s': 'none'}`
  }"
  >
  {{ item.name }}
  </div>
</div>
  <ElButton type="primary" @click="revert">改变顺序</ElButton>
</template>

<style scoped>
.flex{
  display: flex;
  flex-direction: column;
  row-gap: 20px;
}
.item{
  width: 100px;
  height: 50px;
  line-height: 50px;
  background-color: aquamarine;
  border-radius: 10px;
}
</style>
