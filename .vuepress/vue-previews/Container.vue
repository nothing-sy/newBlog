<template>
<div>
<textarea rows="50" cols="20"></textarea>
<span>调整text-area宽度，这行文字会变颜色</span>
</div>
  
</template>
 
<style lang='scss' scoped>
div{
  container-name: box;
  container-type: size;

}

@container box (width > 200px) {
span {
  color: red;
}
}

</style>