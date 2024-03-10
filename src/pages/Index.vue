<template>
  <q-page :class="`flex column relative-position ${isNight && 'night'}`">
    <!-- Location block -->
    <div class="col q-pt-lg q-px-md search">
      <q-input
        @keyup.enter="getWeatherByCity"
        v-model="city"
        placeholder="City"
        dark
        clearable
        outlined
        rounded
      >
        <template v-slot:before>
          <q-btn
            round
            icon="my_location"
            @click="getCurrentLocation"
            color="white"
            text-color="primary"
          />
        </template>
        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherByCity" />
        </template>
      </q-input>
    </div>

    <!-- Block with weather -->
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <img
          :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@4x.png`"
          :alt="weatherData.weather[0].main"
        />
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h1 text-weight-thin">
          {{ Math.round(weatherData.main.temp) }}&deg;
        </div>
        <div class="text-h6 text-weight-medium weather__desc">
          {{ weatherData.weather[0].description }}
        </div>
        <div class="q-mt-md">
          <template v-if="weatherData.rain">
            <div>
              Precipitation:
              <span class="text-weight-medium">{{
                Object.values(weatherData.rain || { k: 0 }).at(0)
              }}</span>
              mm
            </div>
          </template>
          <div>
            Wind:
            <span class="text-weight-medium">{{
              weatherData.wind.speed.toFixed(1)
            }}</span>
            meters per second
          </div>
          <div>
            Wind direction:
            <span class="text-weight-medium">{{ windDirection }}</span>
          </div>
          <div>
            Pressure:
            <span class="text-weight-medium">{{
              weatherData.main.pressure
            }}</span>
            hPa
          </div>
        </div>
      </div>
    </template>

    <!-- Show if no weather data -->
    <template v-else="weatherData">
      <div class="col q-px-sm q-py-lg flex column flex-center">
        <div class="text-h2 text-white text-center">Welcome!</div>
        <div class="text-white text-center q-mt-sm q-mb-lg">
          Click on the below button or search arbitrary city in the search bar
          to see weather results
        </div>
        <q-btn
          color="white"
          text-color="primary"
          icon="my_location"
          label="Search by location"
          class="q-mx-auto block"
          @click="getCurrentLocation"
        />
      </div>
    </template>

    <div class="text-center text-white q-pa-md">Developed by Kumchenko</div>
  </q-page>
</template>

<script>
import axios from "axios";

export default {
  name: "Weather",
  data() {
    return {
      city: "",
      weatherData: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      locationApiUrl: "http://ip-api.com/json/",
      apiKey: process.env.API_KEY,
    };
  },
  methods: {
    async getCurrentLocation() {
      this.$q.loading.show();
      if (this.$q.platform.is.electron) {
        const {
          data: { lat, lon },
        } = await axios.get(this.locationApiUrl);
        this.getWeatherByLocation(lat, lon);
      } else {
        navigator.geolocation.getCurrentPosition(
          ({ coords: { latitude, longitude } }) =>
            this.getWeatherByLocation(latitude, longitude)
        );
      }
    },
    async getWeatherByLocation(latitude, longitude) {
      try {
        const { data } = await axios(this.apiUrl, {
          params: {
            lat: latitude,
            lon: longitude,
            appid: this.apiKey,
            units: "metric",
          },
        });
        this.weatherData = data;
      } catch (e) {
        console.error(e);
      }
      this.$q.loading.hide();
    },
    async getWeatherByCity() {
      this.$q.loading.show();
      try {
        if (this.city) {
          const { data } = await axios(this.apiUrl, {
            params: {
              q: this.city,
              appid: this.apiKey,
              units: "metric",
            },
          });

          this.weatherData = data;
        }
      } catch (e) {
        console.error(e);
      }
      this.$q.loading.hide();
    },
  },
  computed: {
    windDirection() {
      const deg = this.weatherData.wind.deg;
      if (deg >= 337.5 || deg < 22.5) {
        return "North";
      } else if (deg >= 22.5 && deg < 67.5) {
        return "Northeast";
      } else if (deg >= 67.5 && deg < 112.5) {
        return "East";
      } else if (deg >= 112.5 && deg < 157.5) {
        return "Southeast";
      } else if (deg >= 157.5 && deg < 202.5) {
        return "South";
      } else if (deg >= 202.5 && deg < 247.5) {
        return "Southwest";
      } else if (deg >= 247.5 && deg < 292.5) {
        return "West";
      } else if (deg >= 292.5 && deg < 337.5) {
        return "Northwest";
      } else {
        return "Incorrect wind degree";
      }
    },
    isNight() {
      if (this.weatherData) {
        return this.weatherData.weather[0].icon.endsWith("n");
      }
      return false;
    },
  },
};
</script>

<style lang="scss">
.q-page {
  background: #1b3265;
  background: -webkit-linear-gradient(to bottom, #4286f4, #1b3265);
  background: linear-gradient(to bottom, #4286f4, #1b3265);

  &.night {
    background: #373b44;
    background: -webkit-linear-gradient(to bottom, #040404, #373b44);
    background: linear-gradient(to bottom, #040404, #373b44);
  }
}

.search {
  flex-grow: 0;
}

.weather {
  &__desc {
    text-transform: capitalize;
  }
}
</style>
