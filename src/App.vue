<template>
  <div id="app">
    <mapbox v-bind="mapObject"
            @map-init="mapInitialized"
            @map-load="mapLoaded"
            >
    </mapbox>
    <div id="time-travel">
      <h3>Current Year: {{timeTravelYear}}</h3>
      <input type="range" min="2003" max="2017" step="1" v-model="timeTravelYear">
    </div>
  </div>
</template>

<script>
import Mapbox from "mapbox-gl-vue";
import axios from "axios";
export const HTTP = axios.create({
  baseURL: "http://ecs-first-run-alb-1190988938.us-east-2.elb.amazonaws.com/",
  headers: {
    "Access-Control-Allow-Origin": "*"
  },
  auth: {
    username: "adminUser",
    password: "adminPass2018"
  }
});
export default {
  name: "app",
  components: { Mapbox },
  methods: {
    mapInitialized(map) {
      const Geocoder = new MapboxGeocoder({
        accessToken: this.mapObject.accessToken
      });
      map.addControl(Geocoder);
    },
    mapLoaded(map) {
      map.addSource("places", {
        type: "geojson",
        data: this.places
      });
      this.places.features.forEach(function(marker, i) {
        var el = document.createElement("div");
        el.id = "marker-" + i;
        el.className = "marker";
        switch (marker.properties["lob"]) {
          case "DSF":
            el.className += " lob-dsf";
            break;
          case "Property Sales":
            el.className += " lob-property-sales";
            break;
          case "AS":
            el.className += " lob-asset-services";
            break;
        }
        for (var y = 2000; y < 2018; y++) {
          // if (marker.properties["dayOfClosing"].includes(y.toString())) {
          if (marker.properties["dayOfClosing"].substring(0,4) == y.toString()) {
            el.className += " year" + y;
          }
        }

        new mapboxgl.Marker(el, { offset: [0, -15] })
          .setLngLat(marker.geometry.coordinates)
          .addTo(map);
      });
    }
  },
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
      },
      places: {
        type: "FeatureCollection",
        features: []
      },
      timeTravelYear: 2003,
      errors: []
    };
  },
  created() {
    HTTP.get("dealgeolocation")
      .then(response => {
        this.places = response.data;
      })
      .catch(e => {
        this.errors.push(e);
      });
  },
  mounted() {
    console.log(this.timeTravelYear);
    var elementsToShow = document.getElementsByClassName("year2003");
    console.log("year" + this.timeTravelYear);
    for (var i in elementsToShow) {
      if (elementsToShow.hasOwnProperty(i)) {
          elementsToShow[i].classList.add("show-year");
          console.log('if');
        }
        console.log('for');
      }
  },
  watch: {
    timeTravelYear: function (val, oldVal) {
      var elementsToHide = document.getElementsByClassName("year" + oldVal);
      for (var i in elementsToHide) {
        if (elementsToHide.hasOwnProperty(i)) {
          elementsToHide[i].classList.remove("show-year");
        }
      }
      var elementsToShow = document.getElementsByClassName("year" + val);
      for (var i in elementsToShow) {
        if (elementsToShow.hasOwnProperty(i)) {
          elementsToShow[i].classList.add("show-year");
        }
      }
      // document.getElementsByClassName("year" + oldVal).style.display = "none";
      // document.getElementsByClassName("year" + val).style.display = "block";
      // for (var i = 2000; i < 2018; i++) {
      //   if (timeTravelYear == i) {
      //     document.getElementsByClassName("year" + i).style.display = "block";
      //     console.log('changing year ' + i);
      //   }
      // }
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: sans-serif;
  #map {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    .mapboxgl-ctrl-logo {
      display: none;
    }
    .marker {
      opacity: 0;
      border: none;
      cursor: pointer;
      height: 15px;
      width: 15px;
      border-radius: 50%;
      // background-image: url(./assets/marker.png);
      // background-size: 100% 100%;
      // background-color: rgba(0, 0, 0, 0);
      transition: opacity 1s;
      &.lob-dsf {
        background-color: #00a657;
      }
      &.lob-property-sales {
        background-color: #af3cf1;
      }
      &.lob-asset-services {
        background-color: #00b2dd;
      }
      
    }
    .show-year {
      opacity: 1;
    }
  }
  #time-travel {
    position: fixed;
    top: 1em;
    left: 1em;
    width: 20em;
    height: 5em;
    background: #fff;
    border-radius: 0.5em;
    padding: 1em;
    text-align: center;
    text-transform: uppercase;
    input {
      width: 100%;
    }
  }
}
</style>
