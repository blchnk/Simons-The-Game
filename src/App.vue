<template>
  <div id="app">
    <SimonComponent />
  </div>
</template>

<script>
import SimonComponent from './components/SimonComponent.vue'

export default {
  name: 'App',
  components: {
    SimonComponent
  }
}
</script>

<style s>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
