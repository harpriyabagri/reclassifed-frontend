<template>
  <div id="mapContainer" class="basemap"></div>
</template>

<script>
import mapboxgl from "mapbox-gl";

export default {
  name: "BaseMap",
  props: {
    active_filters: Array,
  },
  data() {
    return {
      accessToken:
        "pk.eyJ1IjoiaGFycHJpeWFiYWdyaSIsImEiOiJja2NxeHR6dWgwcjJnMnJtMXhreWN4MWoxIn0.wW12qOAMq730lfuLyXb9nw",
      filter_state: [
        { category: "COVID-19", color: "#f34c46" },
        { category: "Politics", color: "#fa8d4f" },
        { category: "Business", color: "#fdd742" },
        { category: "Sports", color: "#a3e048" },
        { category: "Arts & Media", color: "#49da9a" },
        { category: "Science & Tech", color: "#50d4fe" },
        { category: "Lifestyle", color: "#6073fd" },
        { category: "Community", color: "#ff95d5" },
        { category: "Crisis Updates", color: "#000000" },
      ],
      map: false,
      // filters: this.active_filters,
    };
  },
  watch:{
    active_filters:{
      handler(){
        updateMap()
      },
      deep: true,
    }
  },
  methods: {
    updateMap() {
      map.setLayoutProperty(
        "layer-mypoints",
        "visibility",
        active_filters[i] ? "visible" : "none"
      );
    },
    dashboardClicked() {},

    load() {
      mapboxgl.accessToken = this.accessToken;
      var map = new mapboxgl.Map({
        container: "mapContainer",
        style: "mapbox://styles/harpriyabagri/ckcs1ofh31ejn1iqxzmqwppj2",
        zoom: 1.3,
      });
      this.map = map;

      var nav = new mapboxgl.NavigationControl();
      map.addControl(nav, "top-right");

      map.on("load", function() {
        map.addSource("mypoints", {
          type: "geojson",
          //input the file name of the data you want to display
          //articles.json has one json object for each article
          //grouped.json has one json object for each news outlet
          data: require("./articles.json"),
        });

        map.addLayer({
          id: "layer-mypoints",
          type: "circle",
          source: "mypoints",
          paint: {
            "circle-color": [
              "match",
              ["get", "category"],
              "covid-19",
              "#f34c46",
              "politics",
              "#fa8d4f",
              "business",
              "#fdd742",
              "sports",
              "#a3e048",
              "arts & entertainment",
              "#49da9a",
              "science & tech",
              "#50d4fe",
              "lifestyle",
              "#6073fd",
              "local",
              "#ff95d5",
              "Crisis Updates",
              "#000000",
              /* other */ "#ccc",
            ],
          },
        });

        map.on("click", "layer-mypoints", function(e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var title = e.features[0].properties.title;
          var url = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(
              "<div class=" +
                "title" +
                "> Some Topic </div><a href =" +
                url +
                " target=_" +
                "blank" +
                ">" +
                '"' +
                title +
                '"' +
                "</a>"
            )
            .addTo(map);
        });

        // // Change the cursor to a pointer when the mouse is over the places layer.
        // map.on("mouseenter", "layer-mypoints", function() {
        //   map.getCanvas().style.cursor = "pointer";
        // });
        // Create a popup, but don't add it to the map yet.
        var popup = new mapboxgl.Popup({
          className: "hover-popup",
          closeButton: false,
          closeOnClick: false,
        });

        map.on("mouseenter", "layer-mypoints", function(e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          // Populate the popup and set its coordinates
          // based on the feature found.
          popup
            .setLngLat(coordinates)
            .setHTML("<div> Some Topic </div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "layer-mypoints", function() {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });
      });
    },
  },
  mounted() {
    console.log(this.active_filters);
    this.load();
  },
};
</script>

<style>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css");
.basemap {
  position: absolute;
  width: 100%;
  height: 100%;
}
.hover-popup .mapboxgl-popup-content {
  font-size: 14px;
  background-color: #242424;
  color: rgb(228, 228, 228);
  padding: 2px 8px;
}

.hover-popup .mapboxgl-popup-tip {
  border-top-color: transparent;
  color: transparent;
}

.click-popup .mapboxgl-popup-content {
  background-color: #242424;
  color: rgb(228, 228, 228);
  padding: 10px 15px;
  border-radius: 5px;
}
.click-popup .mapboxgl-popup-content .title {
  font-size: 24px;
  margin: 5px 0px 15px 0px;
}
.click-popup .mapboxgl-popup-tip {
  border-top-color: #242424;
  color: #58585c;
}

.mapboxgl-popup-content a {
  text-decoration: none;
  color: #b1b1b5;
}

.mapboxgl-popup-content a:hover {
  text-decoration: underline;
  color: #d0d0d9;
}

.mapboxgl-popup-close-button {
  font-size: 16px;
  color: white;
}
.mapboxgl-ctrl-group:not(:empty) {
  box-shadow: 0 3px 6px #3535353b;
}
.mapboxgl-ctrl button {
  /*background-color: white; */
  transition: all;
  transition-duration: 150ms;
}
</style>
