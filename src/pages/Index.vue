<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        label="Search"
        dark
        borderless
        @keyup="getWeatherBySearch"
      >
        <template v-slot:prepend>
          <q-icon name="my_location" />
        </template>

        <template v-slot:append>
          <q-icon
            name="search"
            class="cursor-pointer"
            @click="getWeatherBySearch"
          />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="absolute-position text-h4 degree relative-position"
            >&deg;C</span
          >
        </div>
      </div>

      <div class="col text-center">
        '
        <img
          :src="`http://www.openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar<br />
          Weather
        </div>

        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location"></q-icon>
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

var debounce = (cb,delay) => {
     
     var timer;
     return function() {
        if(timer) clearTimeout(timer) 
        var context = this
        var params = arguments
        return setTimeout(() => {
           cb.apply(context,params)
        }, delay);
     }
}

export default defineComponent({
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiKey: "5e9b0ff8f9e747f8ee4f2403e2b6b205",
      apiUrl: "https://api.openweathermap.org/data/2.5/weather?",
    };
  },
  name: "PageIndex",
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show();
      if (this.$q.platform.is.electron) {
        this.$axios.get("https://freegeoip.app/json/").then((response) => {
          this.lon = response.data.longitude;
          this.lat = response.data.latitude;
        });
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          this.lon = position.coords.longitude;
          this.lat = position.coords.latitude;
          this.getWeatherByCoords();
        });
      }
      this.$q.loading.hide();
    },

    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios
        .get(
          `${this.apiUrl}lon=${this.lon}&lat=${this.lat}&appid=${this.apiKey}&units=metric`
        )
        .then((response) => {
         
          this.weatherData = response.data;
        })
        .catch(console.log)
        .finally(() => {
          this.$q.loading.hide();
        });
    },

    getWeatherBySearch: debounce(function() {
      this.$q.loading.show();
      this.$axios
        .get(`${this.apiUrl}q=${this.search}&appid=${this.apiKey}&units=metric`)
        .then((response) => {
          this.weatherData = response.data;
        })
        .catch(console.log)
        .finally(() => {
          this.$q.loading.hide();
        });
    },5000),
  },
});
</script>
<style lang="sass">
.q-page
  background: linear-gradient(to bottom,#136a8a,#267871)
  &.bg-night
    background: linear-gradient(to bottom,#232526,#414345)
  &.bg-day
     background: linear-gradient(to bottom,#00b4db,#0083b0)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url('../assets/img/skyline.png')
  background-size: contain
  background-position: center bottom
</style>
