<template>
  <div>
    <h1 class="text-center">Cum e vremea în...</h1>
    <div class="row justify-center q-gutter-md">
      <div class="input_wrpr col-6">
        <q-select
          ref="SearchInputSelect"
          filled
          use-input
          autofocus
          v-on:keyup="predictCityThrottle"
          label="Introdu locația"
          v-model="selectedCity"
          @input-value="
            val => {
              queryCity = val;
            }
          "
          :options="citySuggestions"
        >
          <!-- <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
                No results
              </q-item-section>
            </q-item>
          </template>-->
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
  OPENWEATHERMAP_API_KEY
} from "../../.env.development.js";
export default {
  name: "Search",
  data: function() {
    return {
      googleMapsApiKey: GOOGLE_MAPS_API_KEY,
      openWeatherMapApiKey: OPENWEATHERMAP_API_KEY,
      queryCity: "",
      citySuggestions: [],
      selectedCity: "",
      currentWeather: {},
      lat: "",
      lon: "",
      sessionToken: undefined
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
      if (this.selectedCity.length > 0) this.getCityCoords();
    },
    "currentWeather.main": function() {
      if (this.currentWeather.main) {
        this.citySuggestions = [];
        this.queryCity = "";
      }
    }
  },
  methods: {
    clear: function() {
      this.selectedCity = "";
      this.queryCity = "";
      this.lat = "";
      this.lon = "";
      this.currentWeather.main = undefined;
      this.citySuggestions = [];
    },

    getData: async function(url = "", params = {}, proxy = false) {
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
        .map(key => {
          return (
            encodeURIComponent(key) + "=" + encodeURIComponent(params[key])
          );
        })
        .join("&");

      const corsAnywhereProxyURL = "https://cors-anywhere.herokuapp.com/";
      let proxiedURL = proxy ? corsAnywhereProxyURL + url : url;
      const proxyHeaders = {
        "Content-Type": "application/json",
        Origin: getHostName(url)
      };

      let request = proxiedURL + "?" + queryString;

      const response = await fetch(request, {
        method: "GET",
        headers: proxy ? proxyHeaders : {},
        redirect: "follow", // manual, *follow, error
        referrerPolicy: "no-referrer" // no-referrer, *client
      });
      return await response.json(); // parses JSON response into native JavaScript objects
    },
    predictCity: async function() {
      const url =
        "https://maps.googleapis.com/maps/api/place/autocomplete/json";
      let data = {
        key: this.googleMapsApiKey,
        input: this.queryCity,
        sessiontoken: this.sessionToken,
        types: "(cities)"
      };
      let json = await this.getData(url, data, true);
      this.citySuggestions = json.predictions.map(city => city.description);
      this.$refs.SearchInputSelect.showPopup();
      return this.citySuggestions;
    },
    predictCityThrottle: _.debounce(function() {
      return this.predictCity();
    }, 100),
    getCityCoords: async function() {
      let params = {
        key: this.googleMapsApiKey,
        address: this.selectedCity
      };

      let url = "https://maps.googleapis.com/maps/api/geocode/json";

      let response = await this.getData(url, params, false);

      let coordsObj = response.results;
      this.lat = coordsObj[0].geometry.location.lat;
      this.lon = coordsObj[0].geometry.location.lng;
    },

    fetchWeather: async function() {
      const url = "https://api.openweathermap.org/data/2.5/weather";
      const params = {
        lat: this.lat,
        lon: this.lon,
        appid: this.openWeatherMapApiKey,
        units: "metric"
      };
      this.currentWeather = await this.getData(url, params, false);
    }
  },
  computed: {},
  created() {
    function generateToken() {
      return (
        Math.random()
          .toString(36)
          .substring(2, 15) +
        Math.random()
          .toString(36)
          .substring(2, 15)
      );
    }
    this.sessionToken = generateToken();
  }
};
</script>

<style scoped>
span.weatherProperty {
  font-weight: bold;
  text-transform: capitalize;
}
</style>
