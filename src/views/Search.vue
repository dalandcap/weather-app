<template>
  <div>
    <h2 class="text-center">What's the weather like in...</h2>
    <div class="row justify-center q-gutter-md">
      <div class="input_wrpr col-6">
        <q-input square filled v-model="input" label="Enter city">
          <template v-if="input" v-slot:append>
            <q-icon
              name="clear"
              @click.stop="input = null"
              class="cursor-pointer"
            />
          </template>
        </q-input>
      </div>
      <div class="list_wrpr col-6">
        <q-list style="width:100% !important">
          <q-item
            @click="fetchWeatherForCity"
            v-for="city in cityPredictions"
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
export default {
  name: "Search",
  data: function() {
    return {
      input: "",
      currentWeather: undefined,
      lat: "",
      lon: "",
      sessionToken: undefined
    };
  },
  watch: {
    input() {
      this.predictCity();
    }
  },
  methods: {
    async predictCity() {
      let key = ***REMOVED***;
      let url =
        "https://maps.googleapis.com/maps/api/place/autocomplete/json?key=" +
        key +
        "&input=" +
        this.input +
        "&sessiontoken=" +
        this.sessionToken;
      let response = await fetch(url);
      if (response.ok) {
        let predictResponse = await response.json();
        console.log(predictResponse);
      }
    },
    async fetchWeather() {
      let apiKey = ***REMOVED***;
      let url =
        "https://api.openweathermap.org/data/2.5/weather?lat=" +
        this.lat +
        "&lon=" +
        this.lon +
        "&appid=" +
        apiKey;
      let response = await fetch(url);
      if (response.ok) {
        this.currentWeather = await response.json();
      }
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
