<template>
  <div id="app">
    <NavBar></NavBar>
    <Map></Map>
  </div>
</template>

<script>
import NavBar from './components/NavBar.vue'
import Map from './components/Map.vue'
export default {
  name: 'App',
  components: {
    NavBar,
    Map
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
  margin-top: 0px;
}
</style>
