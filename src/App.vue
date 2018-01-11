<template>
  <div id="app">
    <mapbox v-bind="mapObject"></mapbox>
  </div>
</template>

<script>
import Mapbox from "mapbox-gl-vue";
export default {
  name: "app",
  components: { Mapbox },
  data() {
    return {
      mapObject: {
        accessToken:
          "pk.eyJ1IjoidGhlamFtZXNsYW5nIiwiYSI6ImNqYmU3c2VtbjJrenkycnBlNzBhM3RxcXgifQ.yFk43uIPRjbjxB6W1_mlJA",
        mapOptions: {
          container: "map",
          style: "mapbox://styles/mapbox/dark-v9",
          center: [-118.2437, 34.0522],
          zoom: 10
        }
      }
    };
  }
};
</script>

<style lang="scss">
#app {
  #map {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
  }
}
</style>
