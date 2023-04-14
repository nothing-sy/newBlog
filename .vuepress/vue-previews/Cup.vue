<template>
    <div class="cup">
    <div class="water">

    </div>

  </div>
  </template>
   
  <script  setup lang='ts'>
  </script>
  <style lang="scss" scoped>
  .cup{
    width: 50px;
    height: 80px;
    background-color: aqua;
    position: relative;
    overflow: hidden;
    clip-path: polygon(-10% 0 ,110% 0, 85% 100%, 15% 100%);
   }
   .water{
    width: 80px;
    height: 80px;
    position: absolute;
    top: -23px;
    left: -10px;
    border-radius: 30%;
    animation: water-loop 4s linear infinite;
    background-color: white;
   }

   @keyframes water-loop {
    from { 
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
   }
  </style>