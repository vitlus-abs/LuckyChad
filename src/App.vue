<template>
  <div id="app">
    <LuckyChad />
  </div>
</template>

<script>
import LuckyChad from './components/LuckyChad.vue';

export default {
  name: 'App',
  components: {
    LuckyChad,
  },
};
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
