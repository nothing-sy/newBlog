<template>
<div>
<textarea rows="5" cols="15"></textarea>
<span>调整text-area宽度，这行文字会变颜色</span>
</div>
  
</template>
 
<style lang='scss' scoped>
div{
  container-name: box;
  container-type: size;
  height: 500px;

}

@container box (width > 200px) {
span {
  color: red;
}
}

</style>