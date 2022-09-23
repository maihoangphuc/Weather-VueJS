<template>
  <!-- bg -->
  <div id="main" :class="isDay ? 'day' : 'night'">
    <!-- app -->
    <div class="app">
      <!-- container -->
      <div class="container weather">

        <!-- title -->
        <h1 class="title text-center">Weather in</h1>
        <!-- end title -->

        <!-- search -->
        <form class="search-location" v-on:submit.prevent="getWeather">
          <input type="text" class="form-control text-muted form-rounded p-4 shadow-sm" placeholder="What City?"
            v-model="citySearch" autocomplete="off" />
        </form>
        <!-- end search -->

        <!-- text no find -->
        <p class="text-center my-3" v-if="cityFound">No city found</p>
        <!-- end text no find -->

        <!-- animation vs card -->
        <div class="card rounded my-3 shadow-lg back-card overflow-hidden" v-if="visible">
          <!-- weather animation -->
          <div>
            <!-- sunny -->
            <div icon="sunny" v-if="clearSky" data-label="Sunny">
              <span class="sun"></span>
            </div>
            <!-- end sunny -->

            <!-- snowy -->
            <div icon="snowy" v-if="snowy" data-label="Chilly">
              <ul>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
              </ul>
            </div>
            <!-- end snowy -->

            <!-- stormy -->
            <div icon="stormy" v-if="stormy" data-label="Soggy">
              <span class="cloud"></span>
              <ul>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
                <li></li>
              </ul>
            </div>
            <!-- end stormy -->

            <!-- cloudy -->
            <div icon="cloudy" v-if="cloudy" data-label="Perfect">
              <span class="cloud"></span>
              <span class="cloud"></span>
            </div>
            <!-- end cloudy -->

            <!-- night moon -->
            <div icon="nightmoon" v-if="clearNight" data-label="Cool!">
              <span class="moon"></span>
              <span class="meteor"></span>
            </div>
            <!-- end night moon -->
          </div>
          <!-- weather animation -->

          <!-- top of card -->
          <div class="card-top text-center" style="margin-bottom: 200px">
            <div class="city-name">
              <div class="city-country">
                <p>{{ weather.cityName }}</p>
                <p>{{ weather.country }}</p>
              </div>
              <div class="date-time text-center">
                <p>{{ time }}</p>
                <p>{{ date }}</p>
              </div>
            </div>
          </div>
          <!-- end top of card -->

          <!-- card middle body -->
          <div class="card-body">
            <!-- card middle -->
            <div class="card-mid">
              <div class="row">
                <div class="col-12 text-center temp">
                  <span>{{ weather.temperature }}&deg;C</span>
                  <p class="my-4">{{ weather.description }}</p>
                </div>
              </div>
              <div class="row">
                <div class="col d-flex justify-content-between px-5 mx-5">
                  <p>
                    <img src="./assets/svg/down.svg" alt="down" />
                    {{ weather.lowTemp }}&deg;C
                  </p>
                  <p>
                    <img src="./assets/svg/up.svg" alt="up" />
                    {{ weather.highTemp }}&deg;C
                  </p>
                </div>
              </div>
            </div>
            <!-- end card middle -->

            <!-- card bottom -->
            <div class="card-bottom px-5 py-4 row">
              <div class="col text-center">
                <p>{{ weather.feelsLike }}&deg;C</p>
                <span>Feels like</span>
              </div>
              <div class="col text-center">
                <p>{{ weather.humidity }}%</p>
                <span>humidity</span>
              </div>
            </div>
            <!-- end card bottom -->
          </div>
          <!-- end card middle body -->
        </div>
        <!-- end animation vs card -->
      </div>
      <!-- end container -->
    </div>
    <!-- end app -->
  </div>
  <!-- end bg -->
</template>

<script>
import moment from 'moment-timezone';

export default {
  data() {
    return {
      cityFound: false,
      visible: false,
      stormy: false,
      cloudy: false,
      clearSky: false,
      clearNight: false,
      snowy: false,
      isDay: true,
      citySearch: "",
      time: "",
      date: "",
      weather: {
        cityName: "Gwarinpa",
        country: "NG",
        temperature: 12,
        description: "Clouds everywhere",
        lowTemp: "19",
        highTemp: "30",
        feelsLike: "20",
        humidity: "55",
      },
    };
  },
  methods: {
    getWeather: async function () {
      const key = process.env.VUE_APP_API_KEY
      const baseURL = `https://openweathermap.org/data/2.5/find?q=${this.citySearch}&appid=${key}&units=metric`;

      //fetch weather
      try {
        const response = await fetch(baseURL);
        const data = await response.json();
        const number = 273.15;
        const newData = data?.list?.map((value) => ({
          name: value?.name,
          country: value?.sys?.country,
          temp: Math.floor(value?.main?.temp - number),
          tempMin: Math.floor(value?.main?.temp_min - number),
          tempMax: Math.floor(value?.main?.temp_max - number),
          feelsLike: Math.floor(value?.main?.feels_like - number),
          humidity: value?.main?.humidity,
          icon: value?.weather[0]?.icon,
          main: value?.weather[0]?.main,
          description: value?.weather[0]?.description,
        }))

        const dt = Object.assign({}, ...newData);

        this.citySearch = "";
        this.weather.temperature = dt?.temp;
        this.weather.cityName = dt?.name;
        this.weather.country = dt?.country;
        this.weather.description = dt?.description;
        this.weather.lowTemp = dt?.tempMin;
        this.weather.highTemp = dt?.tempMax;
        this.weather.feelsLike = dt?.feelsLike;
        this.weather.humidity = dt?.humidity;

        const timeOfDay = dt?.icon;

        ///check for time of day
        if (timeOfDay.includes("n")) {
          this.isDay = false;
        } else {
          this.isDay = true;
        }

        const mainWeather = dt?.main;
        //check weather animations
        if (mainWeather.includes("Clouds")) {
          this.stormy = false;
          this.cloudy = true;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (
          mainWeather.includes("Thunderstorm") ||
          mainWeather.includes("Rain")
        ) {
          this.stormy = true;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clear") && this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = true;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clear") && !this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = true;
          this.snowy = false;
        }
        if (mainWeather.includes("Clouds") && !this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = true;
          this.snowy = false;
        }
        if (mainWeather.includes("Snow")) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = true;
        }
        this.visible = true;
        this.cityFound = false;
      } catch (error) {
        this.cityFound = true;
        this.visible = false;
      }
    },
  },
  watch: {
    "weather.cityName": function () {
      const getTZCountry = moment.tz.zonesForCountry(`${this.weather.country}`)[0];
      const getTZFormat = moment().tz(`${getTZCountry}`);
      const getDate = getTZFormat.format('ll');
      const getTime = getTZFormat.format("LT");
      this.date = getDate;
      this.time = getTime;
    }
  }
};
</script>

<style>
@import "./assets/style/custom.css";
@import "./assets/style/animation.css";
</style>
