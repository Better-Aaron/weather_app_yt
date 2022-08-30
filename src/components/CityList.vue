<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No Locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import { ref } from '@vue/reactivity';
import axios from 'axios';
import router from '../router';
import CityCard from './CityCard.vue';

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
  }

  const requests = [];
  savedCities.value.forEach((city) => {
    requests.push(
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=b166ed1e9e73d246199bd3cf829ae3b0&units=imperial`
      )
    );
  });

  const weatherData = await Promise.all(requests);

  // Flicker Delay
  await new Promise((res) => setTimeout(res, 600));

  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  });
};

await getCities();

const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: { state: city.state, city: city.city },
    query: { id: city.id, lat: city.coords.lat, lng: city.coords.lng },
  });
};
</script>
