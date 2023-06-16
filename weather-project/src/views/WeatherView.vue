<template>
    <div id="app">
        <header>
       <h1>Weather App</h1>
    </header>

       <div class="recent-locations">
        <section v-if="savedLocations.length !== 0" class="searched-locations">
      <div v-for="location in savedLocations" :key="location" class="city-name">
        <button  @click="getWeather(location)" class="location-button">{{ location }}</button>
      </div>
      </section>
    </div>
      


      <div class="search-bar">
         <input type="text" v-model="searchQuery" placeholder="Enter location">
         <button @click="searchWeather">Search</button>
       </div>

      
        <div class="weather-info" v-if="lon !== 0 || lat !== 0">
             <h3 class="location-name">{{ weatherDetails.location }}</h3>
             <section class="temperature-and-description">
             <p class="temperature">{{ weatherDetails.temperature }}°C</p>
             <p class="description">{{ weatherDetails.description }}</p>
             <img :src="weatherDetails.icon" alt="weather-icon">
             </section>

        
             <section class="weather-info-details">
             <p>Humidity: {{ weatherDetails.humidity }}%</p>
             <p>Wind Speed: {{ weatherDetails.wind }} m/s</p>
             <p>Pressure: {{ weatherDetails.pressure }} hPa</p>
             <p>Date and Time: {{ getFormattedDateTime() }}</p>
             </section>
            </div>


            <div class="forecast-container">
                 <h2 class="forecast">Hourly forecast</h2>
          <ul v-for="weatherDetails in weatherForecast" :key=" weatherDetails.time"
            class="forecast-info">
      <img :src="weatherDetails.icon">
      <p class="forecast-detail">{{  weatherDetails.temperature }}°C</p>
      <p class="forecast-detail">{{ getFormattedTime(weatherDetails.time) }} </p>
  </ul>
</div>
       </div>
   </template>  



<script lang="ts">
import axios, { type AxiosResponse } from "axios";
import { defineComponent } from "vue";
import { type WeatherDetails } from "../model/weatherDetails";

export default defineComponent({
  name: "WeatherApp",
  data() {
    return {
      savedLocations: [] as string[],
      searchQuery: "",
      location: "",
      lon: null,
      lat: null,
      weatherDetails: {} as WeatherDetails,
      weatherForecast: [] as WeatherDetails[],
    };
  },
 
  mounted() {
    this.savedLocations = JSON.parse(localStorage.getItem("savedLocations") || "[]");
  },

  methods: {
    getWeather(location: string): void {
      this.fetchWeatherData(location);
      this.storeLocation(location);
    },

    searchWeather(): void {
      if (this.searchQuery.trim() === "") {
        return;
      }
      this.fetchWeatherData(this.searchQuery);
      this.storeLocation(this.searchQuery);
      this.searchQuery = "";
    },

    storeLocation(location: string): void {
      if (this.savedLocations.length === 3) {
        this.savedLocations.pop();
      }
      this.savedLocations.unshift(location);
      localStorage.setItem("savedLocations", JSON.stringify(this.savedLocations));
    },

    resetSearch() {
        this.searchQuery ="";
    },

    getWeatherIconURL(icon: string): string {
      const apiKey = "26e133a61bfa8cd132ef2f59ede3bcb2";
      return `https://openweathermap.org/img/w/${icon}.png?appid=${apiKey}`;
    },

    getFormattedDateTime(): string {
  const date = new Date();
  const options: Intl.DateTimeFormatOptions = {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric',
    second: 'numeric',
    timeZoneName: 'short'
  };
  return date.toLocaleString('en-US', options);
},

    getFormattedTime(dt: number): string {
      const date = new Date(dt * 1000);
      return date.toLocaleTimeString(navigator.language, {
        hour: "2-digit",
        minute: "2-digit",
      });
    },
    
    async fetchWeatherData(location: string): Promise<void> {
      const apiKey = "26e133a61bfa8cd132ef2f59ede3bcb2";
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${location}&appid=${apiKey}&units=metric`;
      try {
        const response: AxiosResponse = await axios.get(url);
        const temperature = Math.round(response.data.main.temp);
        this.weatherDetails = {
          location: response.data.location,
          temperature: temperature,
          description: response.data.weather[0].description,
          time: Date.now(),
          humidity: response.data.main.humidity,
          wind: response.data.wind,
          pressure: response.data.main.pressure,
          icon: this.getWeatherIconURL(response.data.weather[0].icon),
        };
        await this.fetchWeatherForecast(location);
      } catch (error) {
        console.error("Error fetching weather data: ", error);
      }
    },

    async fetchWeatherForecast(location: string): Promise<void> {
      const apiKey = "26e133a61bfa8cd132ef2f59ede3bcb2";
      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${location}&cnt=3&appid=${apiKey}&units=metric`;
      try {
        const response: AxiosResponse = await axios.get(forecastUrl);
        this.weatherForecast = response.data.list.map((forecast: any) => ({
          temperature: Math.round(forecast.main.temp),
          time: forecast.dt,
          icon: this.getWeatherIconURL(forecast.weather[0].icon),
        }));
      } catch (error) {
        console.error("Error fetching weather forecast: ", error);
      }
    },
  },
});
</script>

<style scoped>

header {
  background-color: lightgrey;
  text-align: center;
  padding: 1.5em;
  margin-bottom: 1em;

}
.search-bar {
  margin-bottom: 0.5em;
  width: 100%;
  font-size: 24px;
  padding: 0.25em 0;
  text-align: center;
}

.app {
  padding-left: 2em;
  padding-right: 2em;
}
.searched-locations {
  display: flex;
  justify-content: center;
  gap: 0.5em;
}
.city-name {
  padding: 0em 1em;
  border-radius: 7px;
  margin-bottom: 1.5em;
}
.location-button {
  padding: 0.5em 1em;
}

p {
    font-size: 15px;
}
ul {
    list-style: none;
}

.weather-info {
  padding: 1em 2em 2em 2em;
  border: 1px solid black;
  margin-bottom: 1.5em;
}
.location-name {
  margin-bottom: 1.5em;
  font-size: 25px;
}
.temperature-and-description {
  display: flex;
  justify-content: space-between;
  margin-bottom: 3em;
}
.weather-info-details {
  text-align: center;
  margin-bottom: 2em;
  font-size: 19px;
  display: flex;
  gap: 1em;
}

.forecast {
  text-align: center;
  margin-bottom: 0.25em;
  font-size: 25px;
}
.forecast-container {
  padding: 2em;
  border: 1px solid black;
  margin-bottom: 1.5em;
}
.forecast-info {
  display: flex;
  justify-content: center;
  gap: 30px;
}
.forecast-detail {
  font-size: 20px;
}

.location-name {
  text-align: center;
  margin-bottom: 0.25em;
  font-size: 25px;
}
</style>











   
