<template>
  <div class="depth-map-view">
    <DepthMapDemo />
  </div>
</template>

<script setup>
import DepthMapDemo from '../components/DepthMapDemo.vue'
</script>

<style lang="scss" scoped>
.depth-map-view {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>

