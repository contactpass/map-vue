<template>
  <div id="map">
    <HereMap ref="map"></HereMap>
  </div>
</template>

<script>
// import Map from "./components/Map";
import HereMap from "../components/HereMap.vue";

export default {
  name: "Map",
  components: {
    // Map,
    HereMap,
  },
  mounted() {
  },
};
</script>

<style>
#map {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>