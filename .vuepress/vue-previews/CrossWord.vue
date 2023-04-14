<template>
<div><span>nothing-sy.github.io</span></div>
  
</template>
 
<style lang='scss' scoped>
div{
  background-color: black;
  font-size: 100px;
  font-weight: 500;
  text-align: center;
  filter: contrast(60);/*颜色对比度，让子元素模糊的字体边缘自由选择转黑还是转白*/
}
span{
  color: white;
  filter: blur(2px);
  animation: loop2 4.5s linear infinite;
}

@keyframes loop2 {
  from {
    filter: blur(10px);
    letter-spacing: -50px;
  }
  to{
    letter-spacing: 0px;
    filter: none;
  }
}
</style>