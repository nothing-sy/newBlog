<template>
  <span>
    Documenting web technologies, including CSS, HTML, and JavaScript, since 2005.
  </span>
  </template>
   
  <script  setup lang='ts'>
  </script>
  <style scoped>
  span{
  
    background: linear-gradient(yellow, red, blue);
    background-repeat: no-repeat;
    background-size: 0 5px;
    background-position: right bottom;
    transition: background-size 0.3s linear;
  
  }
  span:hover{
    background-position: left bottom;
    background-size: 100% 5px;
  }
  </style>