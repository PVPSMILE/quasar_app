<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <!-- <q-input @keyup.enter="getWeatherBySearch"  color="white" dark filled v-model="search"
      
      label="Search">
        <template v-slot:prepend>
          <q-icon name="search" color="white" />
          
        </template>
        
      </q-input> -->

          
      <q-input @keyup.enter="getWeatherBySearch"  dark v-model="search" borderless placeholder="Search">

        <template v-slot:before>
          <q-btn
          @click="getLocation" round dense flat icon="my_location" />
        </template>

        <template v-slot:append>
          <q-icon name="search" />
        </template>
      </q-input>
    </div>

    <template v-if="WeatherData"> 
      <div class="col text-white text-center">
        <div class="text-h2 text-weight-light">
          {{ cityName }}
        </div>
        <div class="text-h4 text-weight-light">
          {{ WeatherData.weather[0].main }}
        </div>
        

        <div class="text-h1 text-weight-thin
        q-my-lg ">

         <span><img :src="(`../icons/${WeatherData.weather[0].icon}.png`)" style="max-width: 100px; max-height: 100px;">
            {{ Math.round(WeatherData.main.temp) }}</span>
          <span>&deg;</span>
        </div>

        <div class="text-h5 text-weight-thin
        q-my-lg relative-position">
          <span>feels like {{ Math.round(WeatherData.main.feels_like) }}</span>
          <span>&deg;</span>
        </div>
      </div>
    </template>

    <template v-else> 
      <div class="col text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar<br>Weather
        </div>
        <q-btn
        @click="getLocation"
         class="col" flat>
          <q-icon left size="3em" name="my_location"/>
          <div>Find my location</div>
        </q-btn>
       
      </div>
    </template>
    
    

    <!-- Skyline at the very bottom -->
    <div class="col skyline">

    </div>
  </q-page>
</template>

<script setup>
import { Geolocation } from '@capacitor/geolocation';

defineOptions({
  name: 'IndexPage',
  data() {
    return {
      search: '',
      WeatherData: null,
      lat: null,
      lon: null,
      weather_api_key: "b7c4d03a224739e17e3095f29c1508b7",
      city_api_key: "6ee8b64cd9884bfa92fd9ecf71fec1eb",
    };
  },
  computed: {
    bgClass() {
      if (this.WeatherData){
        if (this.WeatherData.weather[0].icon.endsWith("n")){
          return 'bg-night';
        } else {
          return 'bg-day';
        }
      }

    },
  },
  methods: {
    async getLocation() {
      this.$q.loading.show();
      console.log(this.lat, this.lon);

  

      this.$q.loading.hide();

      if (this.$q.platform.is.electron) {
        const response = await fetch('https://ipinfo.io/json');
        const data = await response.json();
        const loc = data.loc.split(',');
        this.lat = loc[0];
        this.lon = loc[1];
        this.getWeather();

      }
      
      this.$q.loading.hide();

    },
    async getWeather() {
      this.$q.loading.show();

      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.lon}&appid=${this.weather_api_key}&units=metric`
      );
      const response_2 = await fetch(
      `https://api.geoapify.com/v1/geocode/reverse?lat=${this.lat}&lon=${this.lon}&apiKey=${this.city_api_key}`
      );
      const city_data = await response_2.json();
      const weather_data = await response.json();

      this.$q.loading.hide();

      this.cityName = city_data.features[0].properties.city ;  // Название города из ответа API
      this.WeatherData = weather_data;
    },
    async getWeatherBySearch() {
   
    const regex = /^[a-zA-Z\s]*$/;

    if (!regex.test(this.search)) {
      // Если не латиница, показываем ошибку
      this.$q.notify({
        type: 'negative',
        message: 'Пожалуйста, введите только латиницу.'

      });
      return; // Прекращаем выполнение функции
    }

    this.$q.loading.show();

    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${this.search}&appid=${this.weather_api_key}&units=metric`
    );
    console.log(`https://api.openweathermap.org/data/2.5/weather?q=${this.search}&appid=${this.weather_api_key}&units=metric`);
    const weather_data = await response.json();

    this.$q.loading.hide();
    this.WeatherData = weather_data;
    this.cityName = weather_data.name;
  }
  },
});
</script>



<style lang="css">
  .q-page{
    background: #0F2027; 
    background: -webkit-linear-gradient(to right, #2C5364, #203A43, #0F2027); 
    background: linear-gradient(to right, #2C5364, #203A43, #0F2027); 

    color: white;
    min-height: 100vh; /* Ensures the page takes the full viewport height */
    display: flex;
    flex-direction: column;
    justify-content: space-between; /* Pushes skyline to the bottom */
  }
  .q-page.bg-day {
    background: #0ab8d0;
  }
  .q-page.bg-night {
    background: #1a1a1a;
  }

  .q-field__label {
    color: white;
  }

  .skyline {
    flex: 0 0 75px;
    background: url(../assets/skyline.png);
    background-size: contain; /* Stretches image across the width */
    background-position: center bottom; /* Aligns image to the bottom */
    background-repeat: repeat-x;
  }

</style>
