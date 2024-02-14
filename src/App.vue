<template>
  <div id="app">
    <discourse_crawler />
  </div>
</template>

<script>
import discourse_crawler from "./components/discourse_crawler.vue";

export default {
  name: "App",
  components: {
    discourse_crawler,
  },
};
</script>

<style>
html {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  background-color: purple;
}
</style>
