<template>
  <div id="app">
    <list-pokemons/>
  </div>
</template>

<script>
import ListPokemons from './components/ListPokemons.vue';

export default {
  name: 'App',
  components: {
    ListPokemons
  }
}
</script>

<style lang="scss">
@import "~@/assets/scss/vendors/bootstrap-vue/index";
* {
  box-sizing: border-box;
}

html {
  font-family: sans-serif;
}

body {
  margin: 0;
}
</style>
