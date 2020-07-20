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
        ["duders", [22, 100], "www.instagram.com"]
      ]
    };
  },
  mounted() {
    mapboxgl.accessToken = this.accessToken;
    var map = new mapboxgl.Map({
      container: "mapContainer",
      style: "mapbox://styles/harpriyabagri/ckcs1ofh31ejn1iqxzmqwppj2",
      zoom: 1
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
                  "<h5>A northside park that is home to the Lincoln Park Zoo</h5> <a href= 'https://www.google.com' target='_blank'>Click here to read more</a>",
                "marker-size": "medium",
                "marker-color": "#1087bf",
                "marker-symbol": "marker-blue"
              },
              geometry: {
                coordinates: [-123.637596, 49.940403],
                type: "Point"
              },
              id: "7459e13bb6d8ecd1c797e2c168a6ad91"
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
                "marker-symbol": "marker-orange"
              },
              geometry: {
                coordinates: [-123.619185, 49.876367],
                type: "Point"
              },
              id: "acba288f3abd79014145bc16e83fbc78"
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
                "marker-symbol": "marker-green"
              },
              geometry: {
                coordinates: [-123.622554, 49.882534],
                type: "Point"
              },
              id: "f919f7ec1e3bf88e776772311af3b151"
            }
          ]
        }
      });

      map.addLayer({
        id: "layer-mypoints",
        type: "circle",
        source: "mypoints"
      });

      map.on("click", "layer-mypoints", function(e) {
        var coordinates = e.features[0].geometry.coordinates.slice();
        var description = e.features[0].properties.description;

        // Ensure that if the map is zoomed out such that multiple
        // copies of the feature are visible, the popup appears
        // over the copy being pointed to.
        while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
          coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
        }

        new mapboxgl.Popup()
          .setLngLat(coordinates)
          .setHTML(description)
          .addTo(map);
      });

      // Change the cursor to a pointer when the mouse is over the places layer.
      map.on("mouseenter", "layer-mypoints", function() {
        map.getCanvas().style.cursor = "pointer";
      });

      // Change it back to a pointer when it leaves.
      map.on("mouseleave", "layer-mypoints", function() {
        map.getCanvas().style.cursor = "";
      });
    });
  }
};
</script>

<style scoped>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css");
.basemap {
  width: 100%;
  height: 100%;
}
</style>
