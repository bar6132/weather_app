<template>
  <div id="app" :class="typeof weather.main != 'undefined' && weather.main.temp > 16 ? 'warm' : ''">
    <main>
      <div class="search-box">
        <input 
          type="text" 
          name="" 
          id="" 
          class="search-bar" 
          placeholder="Search..."
          v-model="query"
          @keypress="fetchWeather"
        />
      </div>

      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°c</div>
          <div class="weather">{{ weather.weather[0].main }}</div>
        </div>
      </div>

      <div class="forecast" v-if="forecastData">
        <div class="forecast-cards">
        <div v-for="(day, index) in nextFiveDays" :key="index" class="forecast-card">
          <div class="day">{{ getDayOfWeek(day.dt) }}</div>
          <div class="date">{{ getDate(day.dt) }}</div>
          <div class="temp">{{ Math.round(day.main.temp) }}°C</div>
          <div class="weather">{{ day.weather[0].main }}</div>
        </div>
      </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      api_key: process.env.VUE_APP_API_KEY,
      url_base: 'https://api.openweathermap.org/data/2.5/',
      query: '',
      weather: {},
      forecastData: {}
    }
  },
  methods: {
    fetchWeather(e) {
      if (e.key === "Enter") {
        // Fetch current weather
        fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
          .then(res => res.json())
          .then(weatherData => {
            this.weather = weatherData;

            // Fetch forecast data
            fetch(`${this.url_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`)
              .then(res => res.json())
              .then(forecastData => {
                this.forecastData = forecastData;
              })
              .catch(error => {
                console.error('Error fetching forecast data:', error);
              });
          })
          .catch(error => {
            console.error('Error fetching weather data:', error);
          });
      }
    },
    dateBuilder() {
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    },
    getDayOfWeek(timestamp) {
      const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
      const date = new Date(timestamp * 1000);
      return days[date.getDay()];
    },
    getDate(timestamp) {
      const date = new Date(timestamp * 1000);
      return `${date.getDate()}/${date.getMonth() + 1}`;
    }
  },
  computed: {
  nextFiveDays() {
    if (!this.forecastData || !this.forecastData.list) {
      return []; // Return an empty array if forecastData or its list property is undefined
    }
    
    const currentDate = new Date();
    currentDate.setHours(0, 0, 0, 0); // Set to the beginning of the day
    
    // Filter out the forecast data for the current day
    const filteredData = this.forecastData.list.filter(item => {
      const dayDate = new Date(item.dt * 1000);
      dayDate.setHours(0, 0, 0, 0); // Set to the beginning of the day
      return dayDate.getTime() !== currentDate.getTime();
    });
    
    // Group forecast data by day
    const groupedByDay = {};
    for (const item of filteredData) {
      const dayDate = new Date(item.dt * 1000);
      const dateString = `${dayDate.getFullYear()}-${dayDate.getMonth() + 1}-${dayDate.getDate()}`;
      
      if (!groupedByDay[dateString]) {
        groupedByDay[dateString] = item;
      }
    }
    
    // Extract one data point for each day
    const nextFiveDays = Object.values(groupedByDay).slice(0, 5);
    
    return nextFiveDays;
  }
}

}
</script>

<style>
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'montserrat', sans-serif;
}

#app {
  background-image: url('./assets/cold-bg1.jpg');
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}

#app.warm {
  background-image: url('./assets/warm-bg1.jpg');
}

main { 
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;

  appearance: none;
  border:none;
  outline: none;
  background: none;

  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius:0px 16px 0px 16px ;
  transition: 0.4s ;
}

.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.location-box .location {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
  color: #FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}

.weather-box {
  text-align: center;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color:rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;

  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #FFF;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}




.forecast {
  margin-top: 20px;
}

.forecast-cards {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.forecast-card {
  flex-basis: calc(33.33% - 20px);
  margin: 10px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.day {
  font-weight: bold;
  font-size: 18px;
  margin-bottom: 5px;
}

.date {
  font-size: 14px;
  margin-bottom: 10px;
}

.temp {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 10px;
}

.weather {
  font-size: 16px;
  color: #6c757d;
}

</style>