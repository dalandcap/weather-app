<template>
  <div>
    <h1 class="text-center">Cum e vremea în...</h1>
    <div class="row justify-center q-gutter-md">
      <div class="input_wrpr col-6">
        <q-select
          ref="SearchInputSelect"
          filled
          use-input
          clear-icon="close"
          hide-selected
          autofocus
          v-on:keyup="predictCityThrottle"
          label="Introdu locația"
          v-model="selectedCity"
          @input-value="
            (val) => {
              queryCity = val;
            }
          "
          :options="citySuggestions"
        >
        </q-select>
      </div>

      <div class="currentWeather col-6">
        <q-card
          v-if="currentWeather.main"
          square
          class="my-card bg-secondary text-white q-px-lg q-my-xl"
        >
          <q-card-section>
            <div class="text-h2 q-mt-lg q-mb-sm">{{ currentWeather.name }}</div>
            <div class="text-subtitle2">CURRENT WEATHER</div>
          </q-card-section>
          <q-separator dark class="q-my-lg" />
          <q-card-section>
            <p v-for="(value, key) of currentWeather.main" :key="key">
              <span class="weatherProperty">{{ key.replace(/_/g, " ") }}:</span>
              {{ value }}
            </p>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </div>
</template>

<script>
import _ from "lodash";
import {
  GOOGLE_MAPS_API_KEY,
  OPENWEATHERMAP_API_KEY,
} from "../../.env.development.js";

export default {
  name: "Search",
  data: function () {
    return {
      googleMapsApiKey: GOOGLE_MAPS_API_KEY,
      openWeatherMapApiKey: OPENWEATHERMAP_API_KEY,
      queryCity: "",
      citySuggestions: [],
      selectedCity: "",
      currentWeather: {},
      lat: "",
      lon: "",
      sessionToken: "",
      lastSelectedCity: "",
      pastSearches: [],
    };
  },
  watch: {
    lat() {
      if (this.lat && this.lon) this.fetchWeather();
    },
    lon() {
      if (this.lat && this.lon) this.fetchWeather();
    },
    selectedCity() {
      if (this.selectedCity.length > 0) {
        this.getCityCoords();
        this.pastSearches.unshift(this.selectedCity);
        if (this.pastSearches.length > 3) this.pastSearches.pop();
      }
    },
    pastSearches() {
      this.setCookie("past-searches", JSON.stringify(this.pastSearches), 10);
      if (this.citySuggestions.length == 0)
        this.citySuggestions = this.pastSearches;
      this.$refs.SearchInputSelect.showPopup();
    },
    "currentWeather.main": function () {
      if (this.currentWeather.main) {
        this.citySuggestions = [];
        this.queryCity = "";
      }
    },
  },
  methods: {
    clear: function () {
      this.selectedCity = "";
      this.queryCity = "";
      this.lat = "";
      this.lon = "";
      this.currentWeather.main = undefined;
      this.citySuggestions = [];
    },

    getData: async function (url = "", params = {}, proxy = false) {
      function getHostName(url) {
        let match = url.match(/(http|https):\/\/(www[0-9]?\.)?(.[^/:]+)/i);
        if (
          match != null &&
          match.length > 2 &&
          typeof match[2] === "string" &&
          match[2].length > 0
        ) {
          return match[2];
        } else {
          return null;
        }
      }
      // Default options are marked with *
      let queryString = Object.keys(params)
        .map((key) => {
          return (
            encodeURIComponent(key) + "=" + encodeURIComponent(params[key])
          );
        })
        .join("&");

      const corsAnywhereProxyURL = "https://cors-anywhere.herokuapp.com/";
      let proxiedURL = proxy ? corsAnywhereProxyURL + url : url;
      const proxyHeaders = {
        "Content-Type": "application/json",
        Origin: getHostName(url),
      };

      let request = proxiedURL + "?" + queryString;

      const response = await fetch(request, {
        method: "GET",
        headers: proxy ? proxyHeaders : {},
        redirect: "follow", // manual, *follow, error
        referrerPolicy: "no-referrer", // no-referrer, *client
      });
      return await response.json(); // parses JSON response into native JavaScript objects
    },
    predictCity: async function () {
      const url =
        "https://maps.googleapis.com/maps/api/place/autocomplete/json";
      let data = {
        key: this.googleMapsApiKey,
        input: this.queryCity,
        sessiontoken: this.sessionToken,
        types: "(cities)",
      };
      let json = await this.getData(url, data, true);
      this.citySuggestions = json.predictions.map((city) => city.description);
      this.citySuggestions = this.citySuggestions.concat(this.pastSearches);
      this.$refs.SearchInputSelect.showPopup();
    },
    predictCityThrottle: _.debounce(function () {
      return this.predictCity();
    }, 100),
    getCityCoords: async function () {
      let params = {
        key: this.googleMapsApiKey,
        address: this.selectedCity,
      };

      let url = "https://maps.googleapis.com/maps/api/geocode/json";

      let response = await this.getData(url, params, false);

      let coordsObj = response.results;
      this.lat = coordsObj[0].geometry.location.lat;
      this.lon = coordsObj[0].geometry.location.lng;
    },
    fetchWeather: async function () {
      const url = "https://api.openweathermap.org/data/2.5/weather";
      const params = {
        lat: this.lat,
        lon: this.lon,
        appid: this.openWeatherMapApiKey,
        units: "metric",
      };
      this.$refs.SearchInputSelect.blur();
      this.currentWeather = await this.getData(url, params, false);
      this.citySuggestions = this.pastSearches;
    },
    generateToken: function () {
      return "_" + Math.random().toString(36).substr(2, 16);
    },
    setCookie: function (cname, cvalue, exdays) {
      var d = new Date();
      d.setTime(d.getTime() + exdays * 24 * 60 * 60 * 1000);
      var expires = "expires=" + d.toUTCString();
      document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";
    },
    getCookie: function (cname) {
      var name = cname + "=";
      var decodedCookie = decodeURIComponent(document.cookie);
      var ca = decodedCookie.split(";");
      for (var i = 0; i < ca.length; i++) {
        var c = ca[i];
        while (c.charAt(0) == " ") {
          c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
          return c.substring(name.length, c.length);
        }
      }
      return null;
    },
  },
  created() {
    this.sessionToken =
      this.getCookie("sessionToken") ||
      this.setCookie("sessionToken", this.generateToken(), 3);
    // console.log(JSON.parse(this.getCookie("past-searches")));
    this.pastSearches = JSON.parse(this.getCookie("past-searches"));
  },
  beforeDestroy() {
    // this.setCookie("past-searches", JSON.stringify(this.pastSearches), 10);
  },
};
</script>

<style scoped>
span.weatherProperty {
  font-weight: bold;
  text-transform: capitalize;
}
</style>
