<template>
  <div class="app-container">
    <h1>Globe Visualization</h1>
    <GlobeComponent />
  </div>
</template>

<script>
import GlobeComponent from './components/GlobeComponent.vue'

export default {
  name: 'App',
  components: {
    GlobeComponent
  }
}
</script>

<style>
.app-container {
  font-family: Arial, sans-serif;
  text-align: center;
  margin: 0;
  padding: 20px;
}

h1 {
  color: #333;
}
</style>
