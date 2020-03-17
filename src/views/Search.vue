<template>
  <div>
    <h4 class="text-center">Cum e vremea în...</h4>
    <div class="row justify-center q-gutter-md">
      <div class="input_wrpr col-6">
        <q-input
          square
          filled
          v-on:keyup="predictCityThrottle"
          v-model.trim="queryCity"
          label="Enter city"
        >
          <template v-if="queryCity" v-slot:append>
            <q-icon
              name="clear"
              @click.stop="queryCity = ''"
              class="cursor-pointer"
            />
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
        <div v-for="value in currentWeather" :key="value">{{ value }}</div>
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
      currentWeather: undefined,
      lat: "",
      lon: "",
      sessionToken: undefined
    };
  },
  watch: {},
  methods: {
    predictCityThrottle: _.debounce(function() {
      return this.predictCity();
    }, 500),
    async getData(url = "", params = {}) {
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
      let response = await this.getData(url, data);
      return response;
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
