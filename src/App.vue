<script setup lang="ts">
import axios from 'axios';
import { ref } from 'vue';

const appid = '9ea1b8e6e8a329326dde2168fd5ce137';

const city = ref<string>('Москва');
const isLoading = ref(false);
const geo = ref<{ lat: string; lon: string }>({} as { lat: string; lon: string });
const weatherData = ref();

const getGeoCity = async () => {
  isLoading.value = true;
  try {
    const { data } = await axios.get('http://api.openweathermap.org/geo/1.0/direct', {
      params: {
        q: city.value,
        limit: 1,
        appid
      }
    });
    if (!data.length) {
      weatherData.value = null;
      return;
    }
    geo.value = {
      lat: data[0].lat,
      lon: data[0].lon
    };

    const { data: weather } = await axios.get('https://api.openweathermap.org/data/2.5/weather', {
      params: {
        lat: geo.value.lat,
        lon: geo.value.lon,
        units: 'metric',
        lang: 'ru',
        appid
      }
    });
    weatherData.value = weather;
  } catch (err) {
    alert(`Возникла ошибка ${err}`);
  } finally {
    isLoading.value = false;
  }
};

getGeoCity();
</script>

<template>
  <section class="weather">
    <form @submit.prevent="" class="form">
      <input type="text" placeholder="Введите город" v-model="city" />
      <button @click="getGeoCity">Запросить погоду</button>
    </form>
    <div class="weather-data">
      <div class="city">{{ `Город: ${weatherData?.name || city}` }}</div>
      <h2 v-if="isLoading">Loading...</h2>
      <div v-else-if="weatherData" class="weather-params">
        <div>{{ `Температура: ${weatherData.main.temp} градусов` }}</div>
        <div>{{ `Ощущается как: ${weatherData.main.feels_like} градусов` }}</div>
        <div>{{ `Облачность: ${weatherData.weather[0].description}` }}</div>
        <div>{{ `Ветер: ${weatherData.wind.speed} м/c` }}</div>
      </div>
      <div v-else>Данные не найдены!</div>
    </div>
  </section>
</template>

<style scoped>
.weather {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  gap: 50px;
  width: 100%;
}
.form {
  width: 100%;
  display: flex;
  height: 50px;
  input {
    width: 100%;
    outline: none;
    font-size: 20px;
    padding: 5px;
    border-radius: 5px;
  }
  button {
    width: 300px;
    font-size: 20px;
    border-radius: 5px;
    cursor: pointer;
    background-color: #00bd7e;
  }
}
.weather-params,
.city {
  font-size: 20px;
}
.city {
  color: #00bd7e;
}
</style>
