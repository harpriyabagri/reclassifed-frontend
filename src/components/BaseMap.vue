<template>
  <div id="mapContainer" class="basemap"></div>
</template>

<script>
import mapboxgl from "mapbox-gl";

export default {
  name: "BaseMap",
  data() {
    return {
      accessToken:
        "pk.eyJ1IjoiaGFycHJpeWFiYWdyaSIsImEiOiJja2NxeHR6dWgwcjJnMnJtMXhreWN4MWoxIn0.wW12qOAMq730lfuLyXb9nw",
    };
  },
  mounted() {
    mapboxgl.accessToken = this.accessToken;
    var map = new mapboxgl.Map({  
      container: "mapContainer",
      style: "mapbox://styles/harpriyabagri/ckcs1ofh31ejn1iqxzmqwppj2",
      zoom: 1,
    });

    var nav = new mapboxgl.NavigationControl();
    map.addControl(nav, "top-right");
  },
};
</script>

<style scoped>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css");
.basemap {
  width: 100%;
  height: 100%;
}
</style>
