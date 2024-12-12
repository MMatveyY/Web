<template>
  <div id="app">
    <ImageProcessor />
  </div>
</template>

<script>
import ImageProcessor from "./components/ImageProcessor";

export default {
  name: "App",
  components: {
    ImageProcessor,
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
