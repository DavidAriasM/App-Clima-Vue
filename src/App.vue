<template>
  <div id="app">
    <Clima />
  </div>
</template>

<script>
import Clima from './components/Clima.vue'

export default {
  name: 'App',
  components: {
    Clima
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
