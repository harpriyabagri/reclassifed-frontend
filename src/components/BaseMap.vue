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
      map.addSource("mypoints", {
        type: "geojson",
        data: {
          type: "FeatureCollection",
          features: [
            {
              type: "Feature",
              properties: {
                id: "marker-iv1r4bs18",
                title: "Lincoln Park",
                description:
                  "A northside park that is home to the Lincoln Park Zoo",
                "marker-size": "medium",
                "marker-color": "#1087bf",
                "marker-symbol": "marker-blue",
              },
              geometry: {
                coordinates: [-123.637596, 49.940403],
                type: "Point",
              },
              id: "7459e13bb6d8ecd1c797e2c168a6ad91",
            },
            {
              type: "Feature",
              properties: {
                id: "marker-iv1qtkep5",
                title: "Grant Park",
                description:
                  "A downtown park that is the site of many of Chicago's favorite festivals and events",
                "marker-size": "medium",
                "marker-color": "#1087bf",
                "marker-symbol": "marker-orange",
              },
              geometry: {
                coordinates: [-123.619185, 49.876367],
                type: "Point",
              },
              id: "acba288f3abd79014145bc16e83fbc78",
            },
            {
              type: "Feature",
              properties: {
                id: "marker-iv1r541d9",
                title: "Millennium Park",
                description:
                  "A downtown park known for it's art installations and unique architecture",
                "marker-size": "medium",
                "marker-color": "#ffffff",
                "marker-symbol": "marker-green",
              },
              geometry: {
                coordinates: [-123.622554, 49.882534],
                type: "Point",
              },
              id: "f919f7ec1e3bf88e776772311af3b151",
            },
          ],
        },
      });

      map.addLayer({
        id: "layer-mypoints",
        type: "circle",
        source: "mypoints",
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
