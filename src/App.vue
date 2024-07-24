<template>
  <div class="background-image"></div>
  <div class="weather-app" >
    <!-- <div style="background-image: url('assets/img/weather_bg.jpeg'); background-size: cover; background-position: center;"> -->
      

      <header>
      <h1>Weather App</h1>
      <div class="search-container">
        <input type="text" v-model="city" placeholder="Enter a city" />
        <button @click="getWeather" class="search-button">
          <i class="fas fa-search"></i> Search
        </button>
      </div>
    </header>

      <div class="container">
        <div class="row">
          <div class="col-md-8">
            <div class="main-content">
              <div class="weather-info" v-if="weatherData">
  <div class="weather-details-container">
    <div class="weather-details">
      <h2 style="font-size: 25px;">{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
      <div class="weather-icon">
      <img
        :src="`http://openweathermap.org/img/w/${weatherData.weather[0].icon}.png`"
        :alt="weatherData.weather[0].description"
      />
    </div>
      <p class="flex"> <strong>{{ weatherData.weather[0].description }}</strong></p>
      <div class="weather-info-item">
        <span>Temperature:  </span>
        <span> {{ Math.round(weatherData.main.temp) }}°C</span>
      </div>
      <div class="weather-info-item">
        <span>Feels like:</span>
        <span>{{ Math.round(weatherData.main.feels_like) }}°C</span>
      </div>
      <div class="weather-info-item">
        <span>Humidity:</span>
        <span>{{ weatherData.main.humidity }}%</span>
      </div>
      <div class="weather-info-item">
        <span>Wind speed:</span>
        <span>{{ weatherData.wind.speed }} m/s</span>
      </div>
    </div>
    
  </div>
</div>
              <p v-else-if="error" class="error-message">{{ error }}</p>
              <!-- <p v-else class="loading-message">Loading...</p> -->
            </div>
          </div>
          <div class="col-md-4">
            <div class="vertical-line">
            <div class="forecast-container" v-if="uniqueForecastDays.length > 0">
  <h2 class="forecast-header" style="font-size: 25px;">5-Day Forecast</h2>
  <div class="forecast-cards">
    <div class="forecast-card" v-for="(forecast, index) in uniqueForecastDays.slice(0,5)" :key="index">
      <p>{{ formatDate(forecast.dt) }}</p>
      <img
        :src="`http://openweathermap.org/img/w/${forecast.weather[0].icon}.png`"
        :alt="forecast.weather[0].description"
      />
      <p>{{ Math.round(forecast.main.temp) }}°C</p>
      <p>{{ forecast.weather[0].description }}</p>
    </div>
  </div>
</div>
</div>
          </div>

          <div class="col-md-2">
            <div class="vertical-line">
            <div class="search-history-container" v-if="searchHistory.length > 0">
      <h2>Search History</h2>
      <div class="search-history-list">
        <div class="search-history-item" v-for="(city, index) in searchHistory" :key="index">
          {{ city }}
          <button class="delete-button" @click="deleteFromSearchHistory(index)" style=" color: red;">x</button>
        </div>
      </div>
    </div>
  </div>
          </div>
        </div>
      </div>
    </div>
    
  <!-- </div> -->
</template>

<script>
import axios from 'axios';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { faTrashAlt } from '@fortawesome/free-solid-svg-icons'

// using composition api 
export default {
  components: {
    FontAwesomeIcon
  },
  setup() {
		return {
			faTrashAlt,
	  	}
  	},
  data() {
    return {
      city: '',
      weatherData: null,
      forecastData: null,
      error: null,
      searchHistory: []
    };
  },
  computed: {
  uniqueForecastDays() {
    if (!this.forecastData || !this.forecastData.list) return [];
    const uniqueDates = new Set();
    const uniqueForecastDays = [];

    for (const forecast of this.forecastData.list) {
      const date = this.formatDate(forecast.dt);
      if (!uniqueDates.has(date)) {
        uniqueDates.add(date);
        uniqueForecastDays.push(forecast);
      }
    }

    return uniqueForecastDays.slice(0, 5);
  }
},
  methods: {
    async getWeather() {
      try {
        const response = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=metric`
        );
        this.weatherData = response.data;

        const forecastResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=metric`
        );
        this.forecastData = forecastResponse.data;

        // Add the searched city to the search history
        this.addToSearchHistory(this.city);

        this.error = null;
      } catch (error) {
        this.weatherData = null;
        this.forecastData = null;
        this.error = 'Unable to fetch weather data. Please try again.';
      }
    },

    deleteFromSearchHistory(index) {
      this.searchHistory.splice(index, 1);
    },

    addToSearchHistory(city) {
      // Add the city to the beginning of the search history
      this.searchHistory.unshift(city);

      // Keep only the last 5 cities in the search history
      this.searchHistory = this.searchHistory.slice(0, 5);
    },

    formatDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleDateString();
    }
  }
};
</script>

<style>
.vertical-line {
  border-left: 1px solid rgba(255, 250, 250, 0.178);
  height: 85%;
  /* margin: 0 10px; */
}

.search-container {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}

input {
  padding: 8px 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
}

.search-button {
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  font-size: 16px;
  cursor: pointer;
}

.error-message {
  font-size: 18px;
  font-weight: bold;
  color: #ffa600;
  display: grid;
  /* justify-content: center;
  align-items: center;  */
  padding-left: 400px;
  background-color: transparent;
}

.background-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url('assets/img/weather_bg_img.jpeg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  z-index: -1;
  filter: blur(5px);
}

.weather-app {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 80%;
  height: 80%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1;
  background-color: rgba(0, 0, 0, 0.5);
}

header {
  /* padding: 20px; */
  background-color: transparent;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  top: 30%;
  /* margin-bottom: 20px; */
}

.container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  /* padding: 0%; */
  width: 100%;
}

.row {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  width: 90%;
}

.col-md-8 {
  flex: 25%;
  max-width: 20%;
  /* padding-bottom: 0%; */
  
}

.col-md-4 {
  flex: 25;
  max-width: 50%;
  /* padding-bottom: 0%; */
}

.col-md-2 {
  flex: 30%;
  max-width: 20%;
  /* padding-bottom: 0%; */
}

.main-content {
  background-color: transparent;
  width: 50%;
  padding-bottom: 20%;
}

.forecast-container {
  background-color: transparent;
  width: 90%;
  left: 10px;
  padding-top: 3%;
  /* padding: -50px; */
  padding-left: 5%;
}


.weather-info {
  display: flex;
  justify-content: center;
  padding: 10px;
}

.weather-details-container {
  display: flex;
  align-items: center;
  gap: 20px;
}

.weather-details {
  flex: 1;
}

.weather-info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  font-size: 13px;
}

.weather-info-item span {
  margin-right: 8px;
}

.weather-info-item span:last-child {
  margin-right: 0;
}

.weather-icon img {
  width: 55px;
  /* height: 55px; */
  padding-bottom: 0%;
}
.forecast-cards {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 5px;
  /* bottom: 15%; */
}

.forecast-card {
  background-color: transparent;
  padding: 10px;
  text-align: center;
  border-radius: 5px;
  font-size: 12px;
  height: 140px;
}

.forecast-card img {
  max-width: 50px;
}

.forecast-card {
  margin-top: 5px;
}

.search-history-container {
  background-color: transparent;
  padding: 10px;
  /* border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1); */
  width: 250px;
  /* width: 100%;   */
}

.search-history-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.search-history-item {
  background-color: rgba(0, 0, 0, 0.404);
  padding: 5px 10px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 18px;
}

.delete-button {
  cursor: pointer;
  background-color: transparent;  
  border-radius: 100%;
  font-size: 15px;
  padding: 0%;
  padding-left: 10px;
}
</style>