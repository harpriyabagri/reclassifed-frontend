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
        { category: "covid-19", color: "#f34c46", filter: 0 },
        { category: "politics", color: "#fa8d4f", filter: 0 },
        { category: "business", color: "#fdd742", filter: 0 },
        { category: "sports", color: "#a3e048", filter: 0 },
        { category: "arts", color: "#49da9a", filter: 0 },
        { category: "science", color: "#50d4fe", filter: 0 },
        { category: "lifestyle", color: "#6073fd", filter: 0 },
        { category: "local", color: "#ff95d5", filter: 0 },
        { category: "crisis-updates", color: "#000000", filter: 0 },
      ],
      map: null,
      circleColor: ["match", ["get", "category"], "#ccc"],
    };
  },
  watch: {
    active_filters: {
      handler() {
        this.updateMap();
      },
      deep: true,
    },
  },
  methods: {
    updateMap() {
      // console.log(this.active_filters);
      // var i;
      // for (i in this.filter_state) {
      //   this.filter_state[i].filter = this.active_filters[i];
      // }
      // //console.log(this.filter_state)
      // this.map.setLayoutProperty("layer-mypoints", "visibility");

      for (let i in this.active_filters) {
        var temp = JSON.parse(JSON.stringify(this.circleColor));
        if (this.active_filters[i].include) {
          temp.pop();
          temp.push(this.active_filters[i].realCategory);
          temp.push(this.active_filters[i].color);
          temp.push("#ccc");
          console.log(temp);
          let newLayer = {
            id: this.active_filters[i].realCategory,
            type: "circle",
            source: "mypoints",
            paint: {
              "circle-color": temp,
            },
            filter: ["==", "category", this.active_filters[i].realCategory],
          };
          this.map.removeLayer(this.active_filters[i].realCategory);
          this.map.addLayer(newLayer);
        } else {
          if (this.map.getLayer(this.active_filters[i].realCategory)) {
            this.map.removeLayer(this.active_filters[i].realCategory);
          }
        }
      }
    },

    load(map) {
      var nav = new mapboxgl.NavigationControl();
      map.addControl(nav, "top-right");

      map.on("load", function () {
        map.addSource("mypoints", {
          type: "geojson",
          //input the file name of the data you want to display
          //articles.json has one json object for each article
          //grouped.json has one json object for each news outlet
          data: require("./articles.json"),
        });

        // need to duplicate this function so that it applies to every layer (not just covid) and change layer name for each copy
        map.on("click", "covid-19", function (e) {
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
        map.on("click", "local", function (e) {
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
        map.on("mouseenter", "covid-19", function () {
          map.getCanvas().style.cursor = "pointer";
        });
        // Create a popup, but don't add it to the map yet.

        var popup = new mapboxgl.Popup({
          className: "hover-popup",
          closeButton: false,
          closeOnClick: false,
        });

        map.on("mouseenter", "layer-mypoints", function (e) {
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
        map.on("mouseleave", "layer-mypoints", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });
      });
    },
  },

  mounted() {
    mapboxgl.accessToken = this.accessToken;
    this.map = new mapboxgl.Map({
      container: "mapContainer",
      style: "mapbox://styles/harpriyabagri/ckd3fnkh600y21ilxksqkxuj7",
      zoom: 1.3,
    });
    this.load(this.map);
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
