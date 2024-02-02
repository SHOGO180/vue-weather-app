<template>
  <div v-for="city in savedCities" :key="city.id">
    <city-card :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    場所が追加されていません。 場所の追跡を行うには、上記のフォームで検索してください。
  </p>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
  };

  const requests = [];
  savedCities.value.forEach((city) => {
    requests.push(
      axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=8a12db13bdbcc785aa243b57fff9955a&units=metric`)
    )
  });

  const weatherData = await Promise.all(requests);

  // Flicker Delay
  await new Promise((res) => setTimeout(res ,1000))

  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  });
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: {
      state: city.state,
      city: city.city,
    },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
};

</script>