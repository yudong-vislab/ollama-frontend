<template>
  <div id="app">
    <Converse />
  </div>
</template>

<script>
import Converse from './components/Converse.vue'

export default {
  name: 'App',
  components: {
    Converse
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
  margin-top: 60px;
  position: relative;
}
</style>
