<template>
<div>

<text-area></text-area>
<span>调整text-area宽度，这行文字会变颜色</span>
</div>
  
</template>
 
<style lang='scss' scoped>
div{
  container: box size;

}

@container box (width > 200px) {
span {
  color: red;
}
}

</style>