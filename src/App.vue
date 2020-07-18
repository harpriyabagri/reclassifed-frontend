<template>
  <div id="app">
    <BaseMap/>
  </div>
</template>

<script>
import BaseMap from "./components/BaseMap.vue";

export default {
  name: "app",
  components: {
    BaseMap
  },
};
</script>

<style>

#app {
  position: absolute;
  left: 0px;
  top: 0px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  color: #2c3e50;
  height: 100vh;
  width: 100vw;
}
</style>
