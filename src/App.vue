<template>
  <questionComp></questionComp>
</template>

<script>
import questionComp from "./components/questionComp.vue";

export default {
  components: { questionComp },
};
</script>

<style lang="scss">
@import "./styles.scss";
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #ffff;
  background-color: $colour1;
  max-width: 30rem;
  margin: auto;
}
</style>
