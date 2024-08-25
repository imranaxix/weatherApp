<script setup>
import { ref, computed, onMounted } from 'vue';
import bigCities from './majorCities.json';
import SkeletonLoader from './components/SkeletonLoader.vue';  

const api_key = '7da0dfc0fb541ce3a614cab2eaf441c2';
const url_base = 'https://api.openweathermap.org/data/2.5/';
let query = ref('Lahore');
let weather = ref({});
let citySuggestions = ref([]);
let showSuggestions = ref(true);
let isLoading = ref(false);
let errorMessage = ref('');

const majorCities = bigCities.bigCities;

const tempColorClass = computed(() => {
  const temp = weather.value.main?.temp;
  if (temp > 35) return 'text-[#FFD700]';
  if (temp > 25) return 'text-[#FFD700]';
  if (temp > 15) return 'text-[#87CEEB]';
  if (temp > 5) return 'text-[#ADD8E6]';
  return 'text-[#4682B4]';
});

onMounted(() => {
  fetchWeather();
});

const fetchWeather = (e) => {
  if (e?.key === "Enter" || e === undefined) {
    e?.preventDefault();

    if (!query.value.trim()) {
      errorMessage.value = 'Please enter a city name.';
      isLoading.value = false;
      return;
    }

    isLoading.value = true;
    weather.value = {};
    errorMessage.value = ''; 
    console.log(`Fetching weather for ${query.value}`);
    
    fetch(`${url_base}weather?q=${query.value}&units=metric&APPID=${api_key}`)
      .then(res => {
        if (!res.ok) {
          throw new Error('City not found');
        }
        return res.json();
      })
      .then(setResult)
      .catch(error => {
        console.error('Error fetching weather:', error);
        errorMessage.value = 'City not found. Please try another city.';
      })
      .finally(() => {
        isLoading.value = false;
      });

    showSuggestions.value = false;
  }
};

const setResult = (results) => {
  weather.value = results;
};

const fetchCitySuggestions = () => {
  console.log(`Fetching city suggestions for ${query.value}`);
  if (query.value.length >= 1) {
    citySuggestions.value = majorCities
      .filter(city => city.name.toLowerCase().includes(query.value.toLowerCase()))
      .map(city => city.name);
    showSuggestions.value = citySuggestions.value.length > 0;
    console.log('City suggestions:', citySuggestions.value);
    console.log('showSuggestions:', showSuggestions.value);
  } else {
    showSuggestions.value = false;
  }
};

const selectCity = (city) => {
  query.value = city;
  showSuggestions.value = false;
  fetchWeather();
};

const dateBuilder = () => {
  let d = new Date();
  let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
  let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

  let day = days[d.getDay()];
  let date = d.getDate();
  let month = months[d.getMonth()];
  let year = d.getFullYear();

  return `${day} ${date} ${month} ${year}`;
};

const getWeatherIconClass = () => {
  const main = weather.value.weather?.[0]?.main;
  switch (main) {
    case 'Clear': return 'bi bi-sun';
    case 'Clouds': return 'bi bi-cloud';
    case 'Rain': return 'bi bi-cloud-rain';
    case 'Snow': return 'bi bi-cloud-snow';
    case 'Thunderstorm': return 'bi bi-cloud-lightning';
    case 'Drizzle': return 'bi bi-cloud-drizzle';
    case 'Mist':
    case 'Haze': return 'bi bi-cloud-fog';
    case 'Dust': return 'bi bi-cloud-dust';
    case 'Fog': return 'bi bi-cloud-fog';
    case 'Ash': return 'bi bi-cloud-snow';
    case 'Squall': return 'bi bi-cloud-showers';
    case 'Tornado': return 'bi bi-cloud-lightning';
    default: return 'bi bi-cloud';
  }
};

const isClearIconVisible = computed(() => query.value.length >= 1);

const clearQuery = () => {
  query.value = '';
  showSuggestions.value = false;
};

</script>

<template>
  <div class="w-[100wh] h-[100%] bg-black py-5 px-5 md:px-16 min-h-screen text-white">
    <div class="text-[#0FEFFD] text-center text-[48px] py-5 protest-guerrilla-regular">Weather App</div>

    <div class="bg-[#121315] md:px-10 rounded-3xl shadow-2xl px-7">
      <!-- Input with city suggestions -->
      <div class="relative pt-10">
        <i class="bi bi-search absolute left-5 top-[53%] text-black text-2xl"></i>
        <input 
          type="text" 
          placeholder="Search a city..." 
          class="my-10 w-full md:py-4 py-3 px-[70px] rounded-full text-black text-2xl" 
          v-model="query" 
          @input="fetchCitySuggestions" 
          @keydown.enter="fetchWeather"
        >
        <i 
          v-if="isClearIconVisible" 
          class="bi bi-x absolute right-5 top-[53%] cursor-pointer text-black text-2xl" 
          @click="clearQuery"
        ></i>
      </div>
      
      <!-- Suggestions Dropdown -->
      <div class="flex justify-center items-stretch md:px-20">
        <ul v-if="showSuggestions" class="absolute bg-white text-black w-[85%] md:w-[80%] rounded-lg shadow-lg max-h-48 mt-[-30px] overflow-y-auto z-50">
          <li 
            v-for="(city, index) in citySuggestions" 
            :key="index" 
            @click="selectCity(city)" 
            class="px-4 py-2 cursor-pointer hover:bg-gray-200"
          >
            {{ city }}
          </li>
        </ul>
      </div>

      <!-- Skeleton Loader -->
      <SkeletonLoader v-if="isLoading" />

      <!-- Error message for city not found -->
      <div v-if="errorMessage" class="text-center text-red-500 text-2xl mt-4 py-10">
        {{ errorMessage }}
      </div>

      <!-- Weather information display -->
      <div v-else-if="weather.name" class="flex justify-between items-center px-10 py-10 md:flex-row flex-col">
        <div class="flex justify-center items-center flex-col">
          <h3 class="md:text-[50px] text-[40px]">{{ weather.name }} , {{ weather.sys?.country }}</h3>
          <p class="text-[20px]">{{ dateBuilder() }}</p>
        </div>

        <div :class="['flex flex-col justify-center items-end py-8']">
          <div :class="['flex justify-center items-center md:py-0 py-2']">
            <i :class="getWeatherIconClass() + ' md:text-[136px] text-[90px] mr-10'"></i>
            <div class="flex flex-col justify-center items-start">
              <h2 :class="['md:text-[100px] text-[60px]', tempColorClass]">
                {{Math.round(weather.main?.temp) }} <span class="md:ml-[-30px] ml-[-20px]">°</span>
              </h2>
              <p class="text-[30px] mt-[-20px] italic">{{ weather.weather?.[0]?.main }}</p>
            </div>
          </div>
          
          <div class="flex flex-col justify-center items-center">
            <div class="italic md:text-[25px] text-[20px] flex justify-center items-center space-x-4">
            <div class="flex justify-center items-center">
              <p :class="['flex', tempColorClass]">{{Math.round(weather.main?.temp_max) }} <span>°</span></p>
              <p class="px-1">/</p>
              <p :class="['flex', tempColorClass]">{{Math.round(weather.main?.temp_min) }} <span>°</span></p>
            </div>
            <div class="flex justify-between space-x-2 items-center">
              <p>Feels like</p>
              <p :class="['flex', tempColorClass]">{{Math.round(weather.main?.feels_like) }} <span>°</span></p>
            </div>
          </div>
            <div class="flex justify-center items-center italic py-1 md:text-[25px] text-[20px]">
              <i class="bi bi-wind text-[#0FEFFD]"></i>
              <p class="pl-2">{{ Math.round(weather.wind?.speed) }} km/h</p>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<style>
.protest-guerrilla-regular {
  font-family: "Protest Guerrilla", sans-serif;
  font-weight: 400;
  font-style: normal;
}
</style>
