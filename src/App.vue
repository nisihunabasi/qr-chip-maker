<template>
  <div id="container" class="min-h-screen bg-gray-100">
    <h1>QR Code Chip Generator</h1>
    <panel></panel>
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
import panel from './components/panel.vue'

export default {
  name: 'App',
  components: {
    panel
  },
  mounted: function () {
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

</style>
