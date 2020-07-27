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
      for (let i in this.active_filters) {
        var temp = JSON.parse(JSON.stringify(this.circleColor));
        if (this.active_filters[i].include) {
          temp.pop();
          temp.push(this.active_filters[i].realCategory);
          temp.push(this.active_filters[i].color);
          temp.push("#ccc");

          let newLayer = {
            id: this.active_filters[i].realCategory,
            type: "circle",
            source: "mypoints",
            paint: {
              "circle-color": temp,
              "circle-radius": [
                "interpolate",
                ["linear"],
                ["zoom"],
                2,
                ["ln", ["get", "int_sc"]],
                6,
                ["/", ["get", "nat_sc"], 5],
                12,
                ["/", ["get", "prov_sc"], 5],
              ],
            },
            filter: ["==", "category", this.active_filters[i].realCategory],
          };
          if (this.map.getLayer("initial-layer")) {
            this.map.removeLayer("initial-layer");
          }
          if (this.map.getLayer(this.active_filters[i].realCategory)) {
            this.map.removeLayer(this.active_filters[i].realCategory);
          }
          this.map.addLayer(newLayer);
        } else {
          if (this.map.getLayer(this.active_filters[i].realCategory)) {
            this.map.removeLayer(this.active_filters[i].realCategory);
          }
        }
      }
      var actives = false;
      for (let j in this.active_filters) {
        if (this.active_filters[j].include) {
          actives = true;
        }
      }
      if (!actives) {
        if (this.map.isStyleLoaded()) {
          this.map.addLayer({
            id: "initial-layer",
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
              "circle-radius": [
                "interpolate",
                ["linear"],
                ["zoom"],
                2,
                ["ln", ["get", "int_sc"]],
                6,
                ["/", ["get", "nat_sc"], 5],
                12,
                ["/", ["get", "prov_sc"], 3],
              ],
            },
          });
        }
      }
    },

    load(map) {
      var nav = new mapboxgl.NavigationControl();
      map.addControl(nav, "top-right");

      map.on("load", function (e) {
        map.addSource("mypoints", {
          type: "geojson",
          //input the file name of the data you want to display
          //articles.json has one json object for each article
          //grouped.json has one json object for each news outlet
          data: require("./grouped_by_X_2.json"),
        });

        map.addLayer({
          id: "initial-layer",
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

            "circle-radius": [
              "interpolate",
              ["linear"],
              ["zoom"],
              2,
              ["ln", ["get", "int_sc"]],
              6,
              ["/", ["get", "nat_sc"], 5],
              12,
              ["/", ["get", "prov_sc"], 3],
            ],
          },
        });

        var popup = new mapboxgl.Popup({
          className: "hover-popup",
          closeButton: false,
          closeOnClick: false,
        });

        // right now if multiple layers (categories) have a data point in the same location they get stacked, and if you click on the data point it opens up a pop up for every layer that has data point in that location and they stack on top of each other - this should be fixed when we're pulling different data where data points won't be overlapping (hopefully)
        map.on("click", "covid-19", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          //this styling only applies to the covid layer, it needs to be copy/pasted to every other layer as well once you finalize the styling here
          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "covid-19", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "covid-19", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "politics", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "politics", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "politics", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "business", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "business", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "business", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "sports", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "sports", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "sports", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "arts & entertainment", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "arts & entertainment", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "arts & entertainment", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "science", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "science", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "science", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "lifestyle", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "lifestyle", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "lifestyle", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "local", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "local", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "local", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "crisis-updates", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        map.on("mouseenter", "crisis-updates", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "crisis-updates", function () {
          map.getCanvas().style.cursor = "";
          popup.remove();
        });

        map.on("click", "initial-layer", function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;
          var images = e.features[0].properties.image;
          var titles = e.features[0].properties.title;
          var urls = e.features[0].properties.url;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          function jsonToHtml(titles, images, urls) {
            var titles_array = titles.split(",");
            var images_array = images.split(",");
            var urls_array = urls.split(",");

            var beginningstr =
              "<div class='topic'>" +
              topic +
              "</div><div class='headline-wrapper'>";
            var middlestr = "";

            for (var i = 0; i < titles_array.length; i++) {
              if ((i == 0) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("[", "");
                images_array[i] = images_array[i].replace("[", "");
                urls_array[i] = urls_array[i].replace("[", "");
              }
              if ((i == titles_array.length - 1) && (titles_array[i] != null) && (images_array[i] != null) && (urls_array[i] != null)) {
                titles_array[i] = titles_array[i].replace("]", "");
                images_array[i] = images_array[i].replace("]", "");
                urls_array[i] = urls_array[i].replace("]", "");
              }
              middlestr =
                middlestr +
                "<div class='article'><img class='image' src=" +
                images_array[i] +
                "> <br> <a href=" +
                urls_array[i] +
                "target='_blank'>" +
                titles_array[i] +
                "</a></div>";
            }
            var endstr = "</div>";
            return beginningstr + middlestr + endstr;
          }

          var htmlString = jsonToHtml(titles, images, urls);

          new mapboxgl.Popup({ className: "click-popup" })
            .setLngLat(coordinates)
            .setHTML(htmlString)
            .addTo(map);
        });

        // // Change the cursor to a pointer when the mouse is over the places layer.
        // map.on("mouseenter", "covid-19", function () {
        //   map.getCanvas().style.cursor = "pointer";
        // });
        // Create a popup, but don't add it to the map yet.

        map.on("mouseenter", "initial-layer", function (e) {
          // Change the cursor style as a UI indicator.
          map.getCanvas().style.cursor = "pointer";

          var coordinates = e.features[0].geometry.coordinates.slice();
          var topic = e.features[0].properties.topic;

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
            .setHTML("<div>" + topic + "</div>")
            .addTo(map);
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "initial-layer", function () {
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
      style: "mapbox://styles/harpriyabagri/ckd3ksxv805z81lmifjlp8cu1",
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
.headline-wrapper {
  margin: 10px;
  float: left;
}
.article {
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
.image {
  border: 0.1px solid #8DA9BF;
  width: 60px;
  height: 40px;
}
.hover-popup .mapboxgl-popup-content {
  font-size: 13px;
  font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
  background-color: #CBD5DC;
  color: #3F4B55;
  padding: 6px 8px;
  font-weight: bold;
  text-align: center;
  border: 0.1px solid #8da9bf;
}
.hover-popup .mapboxgl-popup-tip {
  border-top-color: transparent;
  color: transparent;
}

.click-popup .mapboxgl-popup-content {
  font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
  background-color: #CBD5DC;
  color: #3F4B55;
  padding: 10px 15px;
  border-radius: 5px;
  border: 0.1px solid #3F4B55;
  width: 300px;
}
.click-popup .mapboxgl-popup-content .title {
  font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
  font-size: 24px;
  /*margin: 5px 0px 15px 0px;*/
}
.click-popup .mapboxgl-popup-tip {
  border-top-color: #CBD5DC;
  color: #CBD5DC;
}

.mapboxgl-popup-content a {
  font: "Avenir";
  text-decoration: none;
  color: #3F4B55;
  margin: 5px 0px 5px 10px;
}

.mapboxgl-popup-content a:hover {
  text-decoration: underline;
  /* color: #d0d0d9; */
  /* color: #cbd5dc; */
}

.mapboxgl-popup-close-button {
  font-size: 16px;
  color: white;
}
.mapboxgl-ctrl-group:not(:empty) {
  background-color: #cbd5dc;
  border: 1px solid #8da9bf;
  border-radius: 5px;
}
.mapboxgl-ctrl button {
  /*background-color: white; */
  color: white;
  transition: all;
  transition-duration: 150ms;
}
.topic {
  font-family: -apple-system, "Avenir", Helvetica, Arial, sans-serif;
  color: #3F4B55;
  text-align: center;
  font-size: 15px;
  font-weight: bold;
}
</style>
