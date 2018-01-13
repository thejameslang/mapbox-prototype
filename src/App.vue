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
      <button v-on:click="playPause = !playPause"><span v-if="!playPause">Play</span><span v-if="playPause">Pause</span></button>
    </div>
    <div class="legend">
      <h6><div class="marker-legend lob-dsf show-year"></div>DSF</h6>
      <h6><div class="marker-legend lob-property-sales show-year"></div>Property Sales</h6>
      <h6><div class="marker-legend lob-asset-services show-year"></div>Asset Services</h6>
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
          if (
            marker.properties["dayOfClosing"].substring(0, 4) == y.toString()
          ) {
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
      errors: [],
      playPause: false
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
        console.log("if");
      }
      console.log("for");
    }
  },
  updated() {
    if (this.playPause) {
      setTimeout(() => {
        if (this.timeTravelYear === 2017) {
          this.timeTravelYear = 2003;
        } else {
          this.timeTravelYear = this.timeTravelYear + 1;
        }
      }, 500);
    }
  },
  watch: {
    timeTravelYear: function(val, oldVal) {
      var elementsToHide = document.getElementsByClassName("year" + oldVal);
      if (!this.playPause) {
        console.log('clearing last year');
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.remove("show-year");
          }
        }
      } else {
        if (oldVal === 2017) {
          console.log('clearing all');
          elementsToHide = document.getElementsByClassName("marker");
          for (var i in elementsToHide) {
            if (elementsToHide.hasOwnProperty(i)) {
              elementsToHide[i].classList.remove("show-year");
            }
          }
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
    },
    playPause: function(val, oldVal) {
      // console.log("playPause");
      // while (val === true) {
      //   if (this.timeTravelYear === 2017) {
      //     this.timeTravelYear = 2003;
      //   } else {
      //     this.timeTravelYear++;
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
  }
  .marker, .marker-legend {
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
  #time-travel {
    position: fixed;
    top: 1em;
    left: 1em;
    width: 20em;
    height: auto;
    background: #fff;
    border-radius: 0.5em;
    padding: 1em;
    text-align: center;
    text-transform: uppercase;
    input {
      width: 100%;
    }
  }
  .legend {
    position: fixed;
    bottom: 1em;
    left: 1em;
    width: 8em;
    height: 4em;
    background: #fff;
    border-radius: 0.5em;
    padding: 0.5em;
    text-transform: uppercase;
    h6 {
      margin: 0.25em 0;
    }
    .marker {
      position: relative;
      top: 3px;
      margin-right: 1em;
      display: inline-block;
    }
  }
}
</style>
