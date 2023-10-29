<template>
  <div class="container bg-blue-400 p-8">
    <div class="input-container flex space-x-4">
      <div class="input-group flex-1">
        <h2 class="text-xl font-semibold mb-2">Введіть назву міста</h2>
        <input
          class="city-input bg-white border border-gray-300 rounded-lg py-2 px-4 w-full"
          v-model="cityInput"
          placeholder="Введіть місто"
          @input="getWeather"
        />
        <button
          class="bg-blue-600 text-white font-medium py-2 px-4 rounded-lg mt-4 hover:bg-blue-700"
          @click="addCity"
        >
          Додати місто
        </button>
      </div>
      <div class="input-group">
        <h2 class="text-xl font-semibold mb-2">Оберіть місто</h2>
        <select
          class="selected-city-select bg-white border border-gray-300 rounded-lg py-2 px-4"
          v-model="selectedCity"
          @change="getSelectedCityWeather"
        >
          <option value="">Оберіть місто</option>
          <option v-for="selectedCity in selectedCities" :key="selectedCity" :value="selectedCity">
            {{ selectedCity }}
          </option>
        </select>
      </div>
    </div>
    <div class="weather-details mt-8 p-4 bg-white rounded-lg">
      <h2 class="text-4xl text-white font-bold mb-4">Погода в  {{ selectedCity }}</h2>
      <div class="weather-details-content">
        <template v-if="selectedCityWeather">
          <div class="weather-info">
            <p class="text-xl">
              <font-awesome-icon class="fa-2x text-blue-600" icon="fa-solid fa-temperature-three-quarters" />
              Температура: {{ selectedCityWeather.main.temp }}°C
            </p>
            <p class="text-xl">
              <font-awesome-icon class="fa-2x text-blue-600" icon="fa-solid fa-cloud" />
              Вологість: {{ selectedCityWeather.main.humidity }}%
            </p>
            <p class="text-xl">
              <font-awesome-icon class="fa-2x text-blue-600" icon="fa-solid fa-water" />
              Тиск: {{ selectedCityWeather.main.pressure }}Па
            </p>
            <p class="text-xl">
              <font-awesome-icon class="fa-2x text-blue-600" icon="fa-solid fa-cloud-sun" />
              Опис погоди: {{ selectedCityWeather.weather[0].description }}
            </p>
            <p class="text-xl">
              <font-awesome-icon class="fa-2x text-blue-600" icon="fa-solid fa-wind" />
              Швидкість вітру: {{ selectedCityWeather.wind.speed }} м/с
            </p>
          </div>
        </template>
        <template v-else>
          <p class="text-lg">Завантаження даних про погоду...</p>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, reactive, onMounted } from "vue";

const cityInput = ref("");
const selectedCity = ref("");
const selectedCities = reactive([]);
const selectedCityWeather = ref(null);

const getWeatherByLocation = (position) => {
  const { latitude, longitude } = position.coords;
  axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=7914d5a440960cfd5df3bd0388a7ad0f&units=metric`)
    .then((response) => {
      selectedCity.value = response.data.name;
      getSelectedCityWeather();
    })
    .catch((error) => {
      console.error("Error fetching weather data by location", error);
    });
};

const getUserLocation = () => {
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(getWeatherByLocation);
  } else {
    console.error("Geolocation is not available in this browser.");
  }
};

const getWeather = async () => {
  if (selectedCity.value) {
    try {
      const response = await axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${selectedCity.value}&appid=7914d5a440960cfd5df3bd0388a7ad0f&units=metric`);
      selectedCityWeather.value = response.data;
    } catch (error) {
      console.error("Error fetching weather data", error);
    }
  }
};

const addCity = () => {
  if (cityInput.value && !selectedCities.includes(cityInput.value)) {
    selectedCities.push(cityInput.value);
  }
  cityInput.value = "";
};

const getSelectedCityWeather = () => {
  if (selectedCity.value) {
    axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${selectedCity.value}&appid=7914d5a440960cfd5df3bd0388a7ad0f&units=metric`)
      .then((response) => {
        selectedCityWeather.value = response.data;
      })
      .catch((error) => {
        console.error("Error fetching weather data for the selected city", error);
        selectedCityWeather.value = null;
      });
  }
};

const showWeather = () => {
  getSelectedCityWeather();
};

onMounted(() => {
 getUserLocation(); 
});
</script>

<style>
body {
    font-family: Arial, sans-serif;
    background-color: #f2f2f2;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
  }

  .container {
    background-color: #7ab2d7;
    padding: 20px;
    border-radius: 8px;
    text-align: center;
  }

  .input-container {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
  }

  .input-group {
    flex: 1;
    padding: 10px;
    background-color: #fff;
    border: 1px solid #ccc;
    border-radius: 6px;
    text-align: left;
  }

  .input-group:first-child{
    margin-right: 5%;
  }

  .input-group h2 {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 10px;
  }

  .city-input {
    width: 90%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 6px;
    background-color: #7ab2d7a1;
  }

  .selected-city-select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 6px;
    background-color: #7ab2d7a1;
  }

  button {
    background-color: #1f618d;
    color: #fff;
    font-weight: bold;
    padding: 10px 20px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    margin-top: 5%;
  }

  button:hover {
    background-color: #154360;
  }

  .weather-details {
    margin-top: 20px;
    padding: 20px;
    background-color: #fff;
    border-radius: 8px;
    text-align: left;
  }

  .weather-details h2 {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 20px;
  }

  .weather-info {
    font-size: 18px;
    margin-bottom: 10px;
  }

  .fa-2x {
    margin-right: 10px;
  }

  .fa-solid{
    color: #154360;
  }
</style>