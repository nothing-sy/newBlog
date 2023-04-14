<template>
  <span>
    Documenting web technologies, including CSS, HTML, and JavaScript, since 2005.
  </span>
  </template>

  <style lang="scss" scoped>
  span{
  
    background: linear-gradient(black);
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