<template>
  <div id="app">
<!--    <img alt="Vue logo" src="./assets/logo.png">-->
<!--    <HelloWorld/>-->
    <GoMoKu></GoMoKu>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import GoMoKu from "@/components/GoMoKu";

export default {
  name: 'App',
  components: {
    GoMoKu
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
}
</style>
