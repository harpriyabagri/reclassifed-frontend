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
      tempData: [
        ["headline example", [12, 12], "www.facebook.com"],
        ["banaas", [22, 22], "www.reddit.com"],
        ["duders", [22, 100], "www.instagram.com"],
      ],
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

    map.on("load", function() {
      map.addLayer({
        id: "historical-places",
        type: "circle",
        source: {
          type: "vector",
          url: "mapbox://harpriyabagri.ab3jqtro",
        },
        "source-layer": "HPC_landmarks-9r0qec",
        paint: {
          "circle-radius": [
            "interpolate",
            ["linear"],
            ["zoom"],
            10,
            ["/", ["-", 2017, ["number", ["get", "Constructi"], 2017]], 30],
            13,
            ["/", ["-", 2017, ["number", ["get", "Constructi"], 2017]], 10],
          ],
          "circle-opacity": 0.8,
          "circle-color": "rgb(171, 72, 33)",
        },
      });
    });
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
