<template>
  <transition name="fade">
    <router-view></router-view>
  </transition>
</template>

<script>
  export default {
  }
</script>

<style>
.el-tooltip__popper {
  max-width: 400px;
  line-height: 180%;
}
</style>
