<template>
  <div>
    <h1 class="text-center">Cum e vremea în...</h1>
    <div class="row justify-center q-gutter-md">
      <div class="input_wrpr col-6">
        <q-input
          square
          filled
          v-on:keyup="predictCityThrottle"
          v-model.trim="queryCity"
          label="Introdu locația"
        >
          <template v-if="queryCity" v-slot:append>
            <q-icon name="clear" @click.stop="clear()" class="cursor-pointer" />
          </template>
        </q-input>
      </div>

      <div class="list_wrpr col-6">
        <q-list style="width:100% !important">
          <q-item
            @click="selectedCity = city"
            v-for="city in citySuggestions"
            :key="city"
            clickable
            v-ripple
          >
            <q-item-section>{{ city }}</q-item-section>
          </q-item>
        </q-list>
      </div>
      <div class="currentWeather col-6">
        <q-card
          v-if="currentWeather.main"
          square
          class="my-card bg-secondary text-white q-px-lg  q-mb-xl"
        >
          <q-card-section>
            <div class="text-h2 q-mt-lg q-mb-md">
              {{ currentWeather.name }}
            </div>
            <div class="text-subtitle2">CURRENT WEATHER</div>
          </q-card-section>

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
export default {
  name: "Search",
  data: function() {
    return {
      queryCity: "",
      citySuggestions: [],
      selectedCity: "",
      currentWeather: { main: undefined },
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
    }
  },
  methods: {
    clear: function() {
      this.selectedCity = "";
      this.queryCity = "";
      this.lat = "";
      this.lon = "";
      this.currentWeather.main = undefined;
    },
    getCityCoords: async function() {
      let coordsObj;
      let key = ***REMOVED***;
      let url =
        "https://maps.googleapis.com/maps/api/geocode/json?key=" +
        key +
        "&address=" +
        this.selectedCity;
      let response = await fetch(url);
      if (response.ok)
        response.json().then(res => {
          coordsObj = res.results;
          this.lat = coordsObj[0].geometry.location.lat;
          this.lon = coordsObj[0].geometry.location.lng;
        });
    },
    predictCityThrottle: _.debounce(function() {
      return this.predictCity();
    }, 500),
    getData: async function(url = "", params = {}) {
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
      let corsAnywhereProxyURL = "https://cors-anywhere.herokuapp.com/";
      let request = corsAnywhereProxyURL + url + "?" + queryString;
      const response = await fetch(request, {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
          Origin: getHostName(url)
        },
        redirect: "follow", // manual, *follow, error
        referrerPolicy: "no-referrer" // no-referrer, *client
      });
      console.log(response);
      return await response.json(); // parses JSON response into native JavaScript objects
    },
    predictCity: async function() {
      const key = ***REMOVED***;
      const url =
        "https://maps.googleapis.com/maps/api/place/autocomplete/json";
      let data = {
        key: key,
        input: this.queryCity,
        sessiontoken: this.sessionToken,
        types: "(cities)"
      };
      let json = await this.getData(url, data);
      console.dir(json);
      this.citySuggestions = json.predictions.map(city => city.description);
      return this.citySuggestions;
    },
    fetchWeather: async function() {
      const apiKey = ***REMOVED***;
      const url = "https://api.openweathermap.org/data/2.5/weather";
      const data = {
        lat: this.lat,
        lon: this.lon,
        appid: apiKey
      };
      this.currentWeather = await this.getData(url, data);
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
