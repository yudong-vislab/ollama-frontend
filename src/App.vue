<template>
  <div id="app">
    <Converse />
    <parse />
  </div>
</template>

<script>
import Converse from './components/Converse.vue'
import parse from './components/parse.vue'
export default {
  name: 'App',
  components: {
    Converse,
    parse,
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
