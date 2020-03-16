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
// countries = JSON.parse(countries);

export default {
  name: "Search",
  data: function() {
    return {
      input: "",
      currentWeather: undefined,
      cities: undefined
    };
  },
  methods: {
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
    cityPredictions() {
      if (this.input) {
        return this.cities.filter(city =>
          city.toLowerCase().startsWith(this.input)
        );
        // return binarySearch(this.cities, this.input);
      }
      return null;
    }
  },
  created() {
    let cities = [];
    for (let country in countries) {
      for (let i = 0; i < country.length; i++) {
        if (countries[country][i]) cities.push(countries[country][i]);
      }
    }
    this.cities = cities;
  }
};
</script>
