<template>
  <div id="app">
    <Header/>
    <mapbox v-bind="mapObject"
            @map-init="mapInitialized"
            @map-load="mapLoaded"
            >
    </mapbox>
    <!-- <div class="data">
      <h3>Square Footage</h3>
      <p>{{ animatedSquareFootage }}</p>
    </div> -->
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
    <div class="filters">
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeApartment"> <h6>Apartment</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeAssistedLiving"> <h6>Assisted Living</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeCondoConstruction"> <h6>Condo - Construction</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeDevSite"> <h6>Dev Site</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeHotel"> <h6>Hotel</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeIndustrial"> <h6>Industrial</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeMiniWarehouse"> <h6>Mini Warehouse</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeMixedUse"> <h6>Mixed Use</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeMultifamily"> <h6>Multifamily</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeOffice"> <h6>Office</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeRetail"> <h6>Retail</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeSeniorsHousingAndCare"> <h6>Seniors Housing &amp; Care</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeSpecialPurpose"> <h6>Special Purpose</h6></div>
      <div class="filter-row"><input type="checkbox" v-model="propertyTypeUnknown"> <h6>Unknown</h6></div>
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
      const Draw = new MapboxDraw();
      map.addControl(Geocoder);
      map.addControl(Draw);
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
        if (!marker.properties["propertyType"]) {
          el.className += " property-type-unknown";
        }
        switch (marker.properties["propertyType"]) {
          case "Multifamily":
            el.className += " property-type-multifamily";
            break;
          case "Assisted Living":
            el.className += " property-type-assisted-living";
            break;
          case "Mini Warehouse":
            el.className += " property-type-mini-warehouse";
            break;
          case "Industrial":
            el.className += " property-type-industrial";
            break;
          case "Retail":
            el.className += " property-type-retail";
            break;
          case "Office":
            el.className += " property-type-office";
            break;
          case "Hotel":
            el.className += " property-type-hotel";
            break;
          case "Mixed Use":
            el.className += " property-type-mixed-use";
            break;
          case "Condo - Construction":
            el.className += " property-type-condo-construction";
            break;
          case "Special Purpose":
            el.className += " property-type-special-purpose";
            break;
          case "Apartment":
            el.className += " property-type-apartment";
            break;
          case "":
            el.className += " property-type-unknown";
            break;
          case "Dev Site":
            el.className += " property-type-dev-site";
            break;
          case "Seniors Housing & Care":
            el.className += " property-type-seniors-housing-and-care";
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

        el.addEventListener("click", function(e) {
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
          var propertyTypeSubCodeName = "";
          var propertyTypeSubCodeValue = "";
          var dayOfClosingName = "";
          var dayOfClosingValue = "";
          var dealValueName = "";
          var dealValueValue = "";
          var propertyTypeName = "";
          var propertyTypeValue = "";
          var netRentableSqFtName = "";
          var netRentableSqFtValue = "";
          var lobName = "";
          var lobValue = "";
          for (var key in marker.properties) {
            var keyName = "";
            if (marker.properties.hasOwnProperty(key)) {
              var keyValue = marker.properties[key];
              if (keyValue.substring(10, 18) == "T00:00:0") {
                keyValue = keyValue.substring(0, 10);
              }

              if (keyValue.toLowerCase() === "null") {
                keyValue = "";
              }

              if (marker.properties[key] === "null") {
                marker.properties[key] = "";
              }

              switch (key) {
                case "propertyTypeSubCode":
                  keyName = "Property Type Sub Code";
                  propertyTypeSubCodeName = keyName;
                  propertyTypeSubCodeValue = keyValue;
                  break;
                case "dayOfClosing":
                  keyName = "Day Of Closing";
                  dayOfClosingName = keyName;
                  dayOfClosingValue = keyValue;
                  break;
                case "dealValue":
                  keyName = "Deal Value";
                  if (keyValue === "") {
                    keyValue = "";
                  } else {
                    keyValue = "$" + Number(keyValue).toLocaleString();
                  }
                  dealValueName = keyName;
                  dealValueValue = keyValue;
                  break;
                case "propertyType":
                  keyName = "Property Type";
                  propertyTypeName = keyName;
                  propertyTypeValue = keyValue;
                  break;
                case "netRentableSqFt":
                  keyName = "Net Rentable Square Feet";
                  this.squareFootage = keyValue;
                  netRentableSqFtName = keyName;
                  if (keyValue != "") {
                    netRentableSqFtValue = Number(keyValue).toLocaleString();
                  }
                  break;
                case "lob":
                  keyName = "Line Of Business";
                  lobName = keyName;
                  if (keyValue === "AS") {
                    keyValue = "Asset Services";
                  }
                  lobValue = keyValue;
                  break;
              }
            }
          }
          if (lobValue != "") {
            setHTMLString +=
              "<h4 class='popup'>" +
              "<span class='property-detail-key'>" +
              lobName +
              "</span>" +
              ": " +
              lobValue +
              "</h4> ";
          }
          if (propertyTypeValue != "") {
            setHTMLString +=
              "<h4 class='popup'>" +
              "<span class='property-detail-key'>" +
              propertyTypeName +
              "</span>" +
              ": " +
              propertyTypeValue +
              "</h4> ";
          }
          if (propertyTypeSubCodeValue != "") {
            setHTMLString +=
              "<h4 class='popup'>" +
              "<span class='property-detail-key'>" +
              propertyTypeSubCodeName +
              "</span>" +
              ": " +
              propertyTypeSubCodeValue +
              "</h4> ";
          }
          if (dealValueValue != "") {
            setHTMLString +=
              "<h4 class='popup'>" +
              "<span class='property-detail-key'>" +
              dealValueName +
              "</span>" +
              ": " +
              dealValueValue +
              "</h4> ";
          }
          if (dayOfClosingValue != "") {
            setHTMLString +=
              "<h4 class='popup'>" +
              "<span class='property-detail-key'>" +
              dayOfClosingName +
              "</span>" +
              ": " +
              dayOfClosingValue +
              "</h4> ";
          }
          if (netRentableSqFtValue != "") {
            setHTMLString +=
              "<h4 class='popup'>" +
              "<span class='property-detail-key'>" +
              netRentableSqFtName +
              "</span>" +
              ": " +
              netRentableSqFtValue +
              "</h4> ";
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
      squareFootage: 354632,
      animatedSquareFootage: 354632,
      timeTravelYear: 2003,
      errors: [],
      playPause: false,
      isActiveDSF: true,
      isActivePropertySales: true,
      isActiveAssetServices: true,
      propertyTypeApartment: true,
      propertyTypeAssistedLiving: true,
      propertyTypeCondoConstruction: true,
      propertyTypeDevSite: true,
      propertyTypeHotel: true,
      propertyTypeIndustrial: true,
      propertyTypeMiniWarehouse: true,
      propertyTypeMixedUse: true,
      propertyTypeMultifamily: true,
      propertyTypeOffice: true,
      propertyTypeRetail: true,
      propertyTypeSeniorsHousingAndCare: true,
      propertyTypeSpecialPurpose: true,
      propertyTypeUnknown: true
    };
  },
  created() {
    HTTP.get("dealgeolocation")
      .then(response => {
        this.places = response.data;
      })
      .catch(e => {
        // this.errors.push(e);
        HTTP.get("dealgeolocation");
      });
  },
  mounted() {
    var elementsToShow = document.getElementsByClassName("year2003");
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
        for (var y = 1999; y < 2018; y++) {
          console.log(y);
          if (y.toString() === val.toString()) {
            console.log('skip');
          } else {
            elementsToHide = document.getElementsByClassName("year" + y.toString());
            console.log(y);
            for (var i in elementsToHide) {
              if (elementsToHide.hasOwnProperty(i)) {
              // if (elementsToHide[i].classList.contains("show-year")) {
                elementsToHide[i].classList.remove("show-year");
              }
            }
          }
        }
      } else {
        if (oldVal === 2017) {
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
            elementsToHide[i].classList.add("hide-lob");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName("lob-dsf");
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-lob");
          }
        }
      }
    },
    isActivePropertySales: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "lob-property-sales"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-lob");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "lob-property-sales"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-lob");
          }
        }
      }
    },
    isActiveAssetServices: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "lob-asset-services"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-lob");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "lob-asset-services"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-lob");
          }
        }
      }
    },
    propertyTypeApartment: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-apartment"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-apartment"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeAssistedLiving: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-assisted-living"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-assisted-living"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeHotel: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-hotel"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-hotel"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeIndustrial: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-industrial"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-industrial"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeMiniWarehouse: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-mini-warehouse"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-mini-warehouse"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeMixedUse: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-mixed-use"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-mixed-use"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeMultifamily: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-multifamily"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-multifamily"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeOffice: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-office"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-office"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeRetail: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-retail"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-retail"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeCondoConstruction: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-condo-construction"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-condo-construction"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeSpecialPurpose: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-special-purpose"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-special-purpose"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeDevSite: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-dev-site"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-dev-site"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeSeniorsHousingAndCare: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-seniors-housing-and-care"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-seniors-housing-and-care"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    propertyTypeUnknown: function(val, oldVal) {
      if (val === false) {
        var elementsToHide = document.getElementsByClassName(
          "property-type-unknown"
        );
        for (var i in elementsToHide) {
          if (elementsToHide.hasOwnProperty(i)) {
            elementsToHide[i].classList.add("hide-property-type");
          }
        }
      } else if (val === true) {
        var elementsToShow = document.getElementsByClassName(
          "property-type-unknown"
        );
        for (var i in elementsToShow) {
          if (elementsToShow.hasOwnProperty(i)) {
            elementsToShow[i].classList.remove("hide-property-type");
          }
        }
      }
    },
    squareFootage: function(newValue, oldValue) {
      var vm = this;
      function animate() {
        if (TWEEN.update()) {
          console.log(this.animatedSquareFootage);
          requestAnimationFrame(animate);
        }
      }

      new TWEEN.Tween({ tweeningNumber: oldValue })
        .easing(TWEEN.Easing.Quadratic.Out)
        .to({ tweeningNumber: newValue }, 500)
        .onUpdate(function() {
          vm.animatedSquareFootage = this.tweeningNumber.toFixed(0);
        })
        .start();

      animate();
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: sans-serif;
  h4.popup {
    text-align: left;
  }
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

  .hide-lob,
  .hide-property-type {
    // display: none;
    opacity: 0 !important;
  }

  .filters {
    font-family: sans-serif;
    text-transform: uppercase;
    position: fixed;
    bottom: 150px;
    left: 10px;
    width: auto;
    height: auto;
    background: #fff;
    padding: 0.5em;
    border-radius: 0.5em;
    h6 {
      display: inline;
    }
  }

  .marker,
  .marker-legend {
    // opacity: 0;
    display: none;
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
    // opacity: 0.5;
    display: block;
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
  .data {
    text-align: center;
    position: fixed;
    bottom: 150px;
    right: 10px;
    width: auto;
    height: auto;
    background: #fff;
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
      cursor: pointer;
      margin: 0.25em 0;
      border-radius: 0.5em;
      padding: 0.5em 0.5em 0.75em 0.5em;
      border-width: 2px;
      border-style: solid;
      border-color: rgba(0, 0, 0, 0);
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
