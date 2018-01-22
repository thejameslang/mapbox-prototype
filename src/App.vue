<template>
  <div id="app">
    <Header/>
    <mapbox v-bind="mapObject"
            @map-init="mapInitialized"
            @map-load="mapLoaded"
            >
    </mapbox>
    <div id="time-travel">
      <h3>Current Year: <span>{{timeTravelYear}}</span></h3>
      <input type="range" min="2003" max="2017" step="1" v-model="timeTravelYear">
      <button v-on:click="playPause = !playPause"><span v-if="!playPause">Play</span><span v-if="playPause">Pause</span></button>
    </div>
    <div class="legend">
      <h6 v-on:click="isActiveDSF = !isActiveDSF" v-bind:class="{ active: isActiveDSF }"><div class="marker-legend lob-dsf show-year"></div>DSF</h6>
      <h6 v-on:click="isActivePropertySales = !isActivePropertySales" v-bind:class="{ active: isActivePropertySales }"><div class="marker-legend lob-property-sales show-year"></div>Property Sales</h6>
      <h6 v-on:click="isActiveAssetServices = !isActiveAssetServices" v-bind:class="{ active: isActiveAssetServices }"><div class="marker-legend lob-asset-services show-year"></div>Asset Services</h6>
    </div>
  </div>
</template>

<script>
import Header from "./components/Header";
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
  components: {
    Mapbox,
    Header
  },
  methods: {
    flyToStore(currentFeature) {
      map.flyTo({
        center: currentFeature.geometry.coordinates,
        zoom: 15
      });
    },
    createPopUp(currentFeature) {
      var popUps = document.getElementsByClassName("mapboxgl-popup");
      if (popUps[0]) popUps[0].remove();

      var popup = new mapboxgl.Popup({ closeOnClick: false })
        .setLngLat(currentFeature.geometry.coordinates)
        .setHTML(
          "<h3>Sweetgreen</h3>" +
            "<h4>" +
            currentFeature.properties.address +
            "</h4>"
        )
        .addTo(map);
    },
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

        el.addEventListener("mouseover", function(e) {
          // map.flyTo({
          //   center: marker.geometry.coordinates,
          //   zoom: 15
          // });

          var popUps = document.getElementsByClassName("mapboxgl-popup");
          if (popUps[0]) popUps[0].remove();

          var popup = new mapboxgl.Popup({
            closeOnClick: false,
            offset: [0, -15]
          }).setLngLat(marker.geometry.coordinates);

          var setHTMLString = "";
          for (var key in marker.properties) {
            var keyName = "";
            if (marker.properties.hasOwnProperty(key)) {
              var keyValue = marker.properties[key];
              if (keyValue.substring(10, 18) == "T00:00:0") {
                keyValue = keyValue.substring(0, 10);
              }

              switch (key) {
                case "propertyTypeSubCode":
                  keyName = "Property Type Sub Code";
                  break;
                case "dayOfClosing":
                  keyName = "Day Of Closing";
                  break;
                case "dealValue":
                  keyName = "Deal Value";
                  keyValue = "$" + keyValue;
                  break;
                case "propertyType":
                  keyName = "Property Type";
                  break;
                case "netRentableSqFt":
                  keyName = "Net Rentable Square Feet";
                  break;
                case "lob":
                  keyName = "Line Of Business";
                  break;
              }

              if (
                marker.properties[key] != "null" &&
                marker.properties[key] != ""
              ) {
                setHTMLString +=
                  "<h4 class='popup'>" +
                  "<span class='property-detail-key'>" +
                  keyName +
                  "</span>" +
                  ": " +
                  keyValue +
                  "</h4> ";
              }
            }
          }
          popup.setHTML(setHTMLString).addTo(map);
        });
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
        // navControl: {
        //   position: 'bottom-left'
        // }
      },
      places: {
        type: "FeatureCollection",
        features: []
      },
      timeTravelYear: 2003,
      errors: [],
      playPause: false,
      isActiveDSF: true,
      isActivePropertySales: true,
      isActiveAssetServices: true
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
      }
    }
    this.playPause = true;
  },
  updated() {
    if (this.playPause) {
      setTimeout(() => {
        if (this.timeTravelYear === 2017) {
          setTimeout(() => {
            this.timeTravelYear = 2003;
          }, 2500);
        } else {
          this.timeTravelYear++;
        }
      }, 500);
    }
  },
  watch: {
    timeTravelYear: function(val, oldVal) {
      var elementsToHide = document.getElementsByClassName("year" + oldVal);
      if (!this.playPause) {
        console.log("clearing last year");
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.remove("show-year");
          }
        }
      } else {
        if (oldVal === 2017) {
          console.log("clearing all");
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
    },
    isActiveDSF: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName("lob-dsf");
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            // elementsToHide[i].classlist.add("hide-lob");
          }
        }
      }
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

  .mapboxgl-ctrl-group {
    margin: 20px 10px 0;
  }

  .mapboxgl-ctrl-attrib {
    display: none;
  }

  .hide-dsf {
    display: none;
  }

  .marker,
  .marker-legend {
    opacity: 0;
    border: none;
    cursor: pointer;
    height: 15px;
    width: 15px;
    border-radius: 50%;
    // background-image: url(./assets/marker.png);
    // background-size: 100% 100%;
    // background-color: rgba(0, 0, 0, 0);
    transition: opacity 0.5s;
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
    opacity: 0.5;
  }
  .popup {
    font-weight: normal;
  }
  .property-detail-key {
    font-weight: bold;
  }

  .mapboxgl-popup-tip,
  .mapboxgl-popup-content {
    background: #fff;
    // border-radius: 0.5em;
    // padding: 1em 0;
    text-align: center;
    text-transform: capitalize;
    border: 1px solid #00a657;
    background-color: #eef8f3;
    color: #1a1a1a;
  }

  #time-travel {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    width: 100%;
    height: auto;
    background: #fff;
    // border-radius: 0.5em;
    padding: 1em 0;
    text-align: center;
    text-transform: capitalize;
    border: 1px solid #00a657;
    background-color: #eef8f3;
    color: #1a1a1a;
    h3 {
      margin: 0;
      font-size: 0.875rem;
      line-height: 1.2;
      color: #1a1a1a;
      margin-bottom: 1em;
      span {
        color: #5a5e64;
      }
    }
    input {
      display: block;
      width: 90%;
      margin: 0 1em;
    }
    button {
      cursor: pointer;
      padding: 0.625rem;
      border: none;
      font-size: 0.875rem;
      font-weight: 700;
      color: #fff;
      background-color: #00a657;
      border-radius: 0.125rem;
      margin-top: 1em;
      box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.25);
      text-decoration: none;
    }
  }
  .legend {
    position: fixed;
    top: 70px;
    left: 10px;
    width: auto;
    height: auto;
    background: #fff;
    border-radius: 0.5em;
    padding: 0.5em;
    text-transform: uppercase;
    display: block;
    .marker-legend {
      background-color: transparent;
      border-width: 2px;
      border-style: solid;
      &.lob-dsf {
        border-color: #00a657;
      }
      &.lob-property-sales {
        border-color: #af3cf1;
      }
      &.lob-asset-services {
        border-color: #00b2dd;
      }
    }
    h6 {
      margin: 0.25em 0;
      border-radius: 0.5em;
      padding: 0.5em 0.5em 0.75em 0.5em;
      border-width: 2px;
      border-style: solid;
      border-color: rgba(0,0,0,0);
      &.active {
        &:first-child {
          border-color: #00a657;
          .marker-legend {
            background-color: #00a657;
          }
        }
        &:nth-child(2) {
          border-color: #af3cf1;
          .marker-legend {
            background-color: #af3cf1;
          }
        }
        &:last-child {
          border-color: #00b2dd;
          .marker-legend {
            background-color: #00b2dd;
          }
        }
      }
    }
    .marker-legend {
      position: relative;
      top: 3px;
      margin-right: 1em;
      display: inline-block;
      opacity: 1;
    }
  }
}
</style>
