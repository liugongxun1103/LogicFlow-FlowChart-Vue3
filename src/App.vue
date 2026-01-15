<template>
  <div class="container">
    <Diagram />
  </div>
</template>

<script setup>
import Diagram from './components/Diagram.vue'
</script>

<style>
html, body {
  height: 100%;
}
body {
  padding: 0;
  margin: 0;
}
</style>
<style scoped>
.container {
  background-color: #f8f9fa;
  height: 100%;
}
</style>
