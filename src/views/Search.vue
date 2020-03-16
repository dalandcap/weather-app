<template>
  <div>
    <h2 class="text-center">What's the weather like in...</h2>
    <div class="row justify-center q-gutter-md">
      <div class="input_wrpr col-6">
        <q-input square filled v-model="input" label="Enter city">
          <template v-if="input" v-slot:append>
            <q-icon name="clear" @click.stop="input = null" class="cursor-pointer" />
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
import countries from "../assets/countries.json";

export default {
  name: "Search",
  data: function() {
    return {
      input: undefined,
      currentWeather: undefined,
      cityPredictions: [],
      countriesString: countries
    };
  },
  methods: {
    // predictCities() {
    //   for (let country of this.countriesMemory) {
    //     console.log(country);
    //   }

    // for (let [country, cities] of Object.entries(this.countries)) {
    //   console.log(country, cities);
    // }

    // predictCitiesOld() {
    //   for (let country in this.countries)
    //     for (let city in country) {
    //       if (city == this.input || city.startsWith(this.input)) {
    //         // this.cityPredictions.push(city);
    //         console.log(city);
    //       }
    //     }
    // for (let i = 0; i < countries.length; i++)
    //   for (let j = 0; j < countries.length; i++) {
    //     if (city == this.input || city.startsWith(this.input)) {
    //       this.cityPredictions.push(city);
    //     }
    // }
    // },
    async fetchWeatherForCity() {
      let apiKey = ***REMOVED***;
      let url =
        "https://api.openweathermap.org/data/2.5/weather?q=" +
        this.input +
        "&appid=" +
        apiKey;
      let response = await fetch(url);
      if (response.ok) {
        this.currentWeather = await response.json();
      }
    }
  },
  computed: {
    // countriesMemory() {
    //   return JSON.parse(this.countriesString);
    // }
    // cityPredictions() {
    //   return ["unu", "doi", "trii", "undefined"];
    // }
  },
  watch: {
    input() {
      // this.predictCities();
    }
  },
  created() {
    console.log(JSON.parse(this.countriesString));
  }
};
</script>
