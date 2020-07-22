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
        "pk.eyJ1IjoiaGFycHJpeWFiYWdyaSIsImEiOiJja2NxeHR6dWgwcjJnMnJtMXhreWN4MWoxIn0.wW12qOAMq730lfuLyXb9nw"
    };
  },
  methods: {
    load() {
      mapboxgl.accessToken = this.accessToken;
      var map = new mapboxgl.Map({
        container: "mapContainer",
        style: "mapbox://styles/harpriyabagri/ckcs1ofh31ejn1iqxzmqwppj2",
        zoom: 1.3
      });

      var nav = new mapboxgl.NavigationControl();
      map.addControl(nav, "top-right");

      map.on("load", function() {
        map.addSource("mypoints", {
          type: "geojson",
          //input the file name of the data you want to display
          //articles.json has one json object for each article
          //grouped.json has one json object for each news outlet
          data: require("./articles.json")
        });

        map.addLayer({
          id: "layer-mypoints",
          type: "circle",
          source: "mypoints"
        });

        map.on("click", "layer-mypoints", function(e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var outlet = e.features[0].properties.outlet;
          var location = e.features[0].properties.location;
          // var title = e.features[0].properties.title;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML("<h3>" + outlet + "</h3><h3>" + location + "</h3>")
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
  },
  mounted() {
    this.load();
  }
};
</script>

<style>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css");
.basemap {
  width: 100%;
  height: 100%;
}
.click-popup .mapboxgl-popup-content {
  background-color: #58585c;
  color: white;
}
.click-popup .mapboxgl-popup-tip {
  border-top-color: #58585c;
  color: #58585c;
}

.mapboxgl-popup-content a {
  text-decoration: none;
  color: #b1b1b5;
}

.mapboxgl-popup-content a:hover {
  text-decoration: none;
  color: #d0d0d9;
}

.mapboxgl-popup-close-button {
  color: white;
}
</style>
