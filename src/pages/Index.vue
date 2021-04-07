<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location"/>
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search"/>
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
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" alt="">
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location"/>
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col skyline">

    </div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: '',
      weatherData: null,
      lat: null,
      lon: null,
      apiKey: process.env.OPEN_WEATHER_MAP_API_KEY
    }
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show()

      if (this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/').then(response => {
          this.lat = response.data.latitude
          this.lon = response.data.longitude
          this.getWeatherByCoords()
          this.$q.loading.hide()
        })
      } else {
        navigator.geolocation.getCurrentPosition(position => {
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()

        })
      }
      this.$q.loading.hide()
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(`https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    },
    getWeatherBySearch() {
      if (!this.search) return

      this.$q.loading.show()
      this.$axios(`https://api.openweathermap.org/data/2.5/weather?q=${this.search}&appid=${this.apiKey}&units=metric`).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    }
  }
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to right, #2C5364, #203A43, #0F2027)

  &.bg-night
    background: linear-gradient(to top, #434343, #000000)

  &.bg-day
    background: linear-gradient(to left, #91EAE4, #86A8E7, #7F7FD5)

.degree
  top: -44px

.skyline
  flex: 0 0 100px
  background: url('../assets/skyline.svg')
  background-size: contain
  background-position: center bottom
</style>
