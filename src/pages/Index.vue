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
          <q-icon
            @click="getLocation"
            name="my_location"
          />
        </template>

        <template v-slot:append>
          <q-btn
            @click="getWeatherBySearch"
            icon="search"
            dense
            flat
            round
          />
        </template>
      </q-input>
    </div>
    <template>
      <div class="text-white text-center q-mb-lg">
        {{ date }}
      </div>
    </template>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light q-mt-md">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
        <div class="text-h8 text-weight-thin q-my-md relative-position">
          <span>Feels like {{ Math.round(weatherData.main.feels_like) }}</span>
          <span class="text-h8 relative-position degree-small">&deg;C</span>
        </div>
        <div class="text-h8 text-weight-light q-mt-md">
          wind {{ Math.round(weatherData.wind.speed) }} m/s
        </div>
      </div>
      <div class="col text-center">
        <img :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`">
      </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Weather
        </div>
        <q-btn
          @click="getLocation"
          class="col"
          flat
        >
          <q-icon left size="3em"
            name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import { Plugins } from '@capacitor/core'
import { format } from 'date-fns'

const { Geolocation } = Plugins

export default {
  name: 'PageIndex',
  data() {
    return {
      date: '',
      search: '',
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: '650419ad3b154930d71224e1a074ad50'
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
    getDate() {
      this.date = format(new Date(), 'MMMM d, H:mm:ss')
      setTimeout(this.getDate, 1000)
    },
    getLocation() {
      this.$q.loading.show()

      if (this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/').then(response => {
          this.lat = response.data.latitude
          this.lon = response.data.longitude
          this.getWeatherByCoords()
        })
      } else {
        Geolocation.getCurrentPosition().then(position => {
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()
        })
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(`${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    },
    getWeatherBySearch() {
      this.$q.loading.show()
      this.$axios(`${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    }
  },
  mounted() {
    this.getDate()
  }
}
</script>

<style lang="sass">
  .q-page
    background: linear-gradient(to bottom, #136a8a, #267871)
    &.bg-night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #00b4db, #00B3b0)
  .degree
    top: -44px
  .degree-small
    top: 0px
  .skyline
    flex: 0 0 100px
    background: url(/skyline.png)
    background-size: contain
    background-position: center bottom
</style>