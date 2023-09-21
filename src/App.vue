<template>
  <TemplateHome/>
</template>

<script>
  import TemplateHome from './pages/TemplateHome.vue'

  export default {
    name: 'App',
    components: {
      TemplateHome
    }
  }
</script>

<style>
  #app {
    margin-top: 60px;
    padding: 10px;
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    font-size: 15px;
    color: #2c3e50;
  }
</style>
