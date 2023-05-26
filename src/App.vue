<template>
  <div>
    <h1>Weather Search</h1>

    <div class="search-locations">
      <div v-for="location in lastSearchLocations" :key="location" class="location">{{ location }}&nbsp;</div>
    </div>

    <div class="search">
      <input id="searchInput" v-model="searchQuery" type="text" placeholder="Search location" @blur="searchWeather"
             @input="handleSearchInput"/>
    </div>
    <div v-if="weatherData" class="weather-panel">
      <div class="weather-info">
        <div class="line1">
          <div class="weather-icon">
            <img :src="getWeatherIconUrl(weatherData.weather[0].icon)" alt="Weather Icon" />
          </div>
          <div class="temperature">{{ weatherData.main.temp }}°C</div>
        </div>
        <div class="line2">{{ weatherData.weather[0].description }}</div>
        <div class="line3 additional-info">
          <div>Humidity: {{ weatherData.main.humidity }}%, Wind Speed: {{ weatherData.wind.speed }} m/s</div>
          <!-- Add more additional weather information as needed -->
        </div>
      </div>


    </div>

    <h2>Hourly Forecast</h2>
    <div class="hourly-forecast">
      <div v-for="forecast in hourlyForecast" :key="forecast.time" class="forecast-item">
        <div class="forecast-icon">
          <img :src="getWeatherIconUrl(forecast.icon)" alt="Weather Icon"/>
        </div>
        <div class="forecast-info">
          <div class="temperature">{{ forecast.temperature }}°C</div>
          <div class="time">{{ forecast.time }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {defineComponent, ref} from 'vue';
import axios from 'axios';

export default defineComponent({
  data() {
    return {
      searchQuery: '',
      weatherData: null,
      lastSearchLocations: [],
      hourlyForecast: null,
    };
  },
  setup() {
    const searchTimer = ref(null);

    const handleSearchInput = () => {
      clearTimeout(searchTimer.value);
      searchTimer.value = setTimeout(() => {
        this.searchWeather();
      }, 2000);
    };

    return {
      handleSearchInput,
    };
  },

  methods: {
    async searchWeather() {
      try {
        const response = await axios.get(
            `https://api.openweathermap.org/data/2.5/weather?q=${this.searchQuery}&units=metric&limit=1&appid=6bf6b2b8fda4fa0d4158d2fb08ec618a`
        );
        this.weatherData = response.data;
        this.saveSearchLocation(this.searchQuery);

        const hourlyResponse = await axios.get(
            `https://api.openweathermap.org/data/2.5/forecast?lat=${this.weatherData.coord.lat}&lon=${this.weatherData.coord.lon}&units=metric&limit=1&appid=6bf6b2b8fda4fa0d4158d2fb08ec618a`
        );
        this.hourlyForecast = this.extractHourlyForecast(hourlyResponse.data.list);
        this.searchQuery = ''; // Clear the search input after search
      } catch (error) {
        console.error('Error fetching weather data:', error);
      }
    },
    saveSearchLocation(location: string) {
      this.lastSearchLocations.unshift(location);
      if (this.lastSearchLocations.length > 3) {
        this.lastSearchLocations.pop();
      }
    },
    extractHourlyForecast(forecastData: any[]): any[] {
      return forecastData
          .filter((forecast, index) => index % 5 === 0)
          .map(forecast => ({
            time: forecast.dt_txt,
            temperature: forecast.main.temp,
            icon: forecast.weather[0].icon,
          }));
    },
    getWeatherIconUrl(iconCode: string): string {
      return `https://openweathermap.org/img/w/${iconCode}.png`;
    },
  },
});
</script>

<style scoped>
/* Add custom styles as needed */
.search {
  align-self: center;
  Width: 500px;
  align-items: center;
  margin-bottom: 1rem;
}

.weather-panel {
  align-items: center;
  margin-bottom: 1rem;
}

.weather-icon img {
  width: 64px;
  height: 64px;
}

.search-locations {
  width: 500px;
  display: flex;
  margin: 2rem;
}


.forecast-item {
  display: flex;
  align-items: center;
  margin-right: 1rem;
  margin-bottom: 0.5rem;
}

.forecast-icon img {
  width: 32px;
  height: 32px;
  margin-right: 0.5rem;
}

.forecast-info {
  display: flex;
  align-items: center;
}

.forecast-info .temperature {
  margin-right: 0.5rem;
}

.weather-info {
  width: 500px;
  display: flex;
  flex-direction: column;
  border: 2px solid black;
  padding: 1rem;
  align-self: center;
}

.hourly-forecast{
  width: 500px;
  display: flex;
  flex-direction: column;
  border: 2px solid black;
  padding: 1rem;
}

.line1 {
  display: flex;
  align-items: center;
}

.weather-icon img {
  width: 64px;
  height: 64px;
}

.temperature {
  margin-left: 0.5rem;
}

.line2 {
  margin-top: 0.5rem;
}

.line3 {
  margin-top: 0.5rem;
}

.additional-info div {
  margin-bottom: 0.5rem;
}
</style>