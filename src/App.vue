<template>
  <div id="app">
    <GridPage></GridPage>
  </div>
</template>

<script>
import GridPage from './components/GridPage.vue';

export default {
  name: 'App',
  components: {
    GridPage
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
