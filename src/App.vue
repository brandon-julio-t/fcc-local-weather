<script lang="ts">
import { defineComponent } from "vue";
import axios from "axios";
import IWeather from "./contracts/IWeather";

interface IData {
  isLoading: boolean;
  weather: IWeather | null;
  countryCode: string;
  countryName: string;
  temperature: number;
}

interface IResponse {
  weather: IWeather[];
  main: { temp: number };
  sys: { country: string };
  name: string;
}

export default defineComponent({
  async mounted() {
    this.isLoading = true;

    const position = await new Promise<GeolocationPosition>((resolve) => {
      navigator.geolocation.getCurrentPosition(resolve);
    });

    const { longitude, latitude } = position.coords;

    const apiUrl = "https://weather-proxy.freecodecamp.rocks/api/current";
    const { data } = await axios.get<IResponse>(apiUrl, {
      params: { lon: longitude, lat: latitude },
    });

    this.weather = data.weather[0];
    this.countryCode = data.sys.country;
    this.countryName = data.name;
    this.temperature = data.main.temp;

    this.isLoading = false;
  },

  data(): IData {
    return {
      isLoading: false,
      weather: null,
      countryCode: "",
      countryName: "",
      temperature: 0,
    };
  },
});
</script>

<template>
  <main class="container mx-auto h-screen grid place-items-center">
    <section class="border-slate-200 shadow hover:shadow-md transition-all rounded-xl p-6">
      <h1 class="text-4xl text-center mb-4">Local Weather</h1>

      <div v-if="isLoading" class="animate-pulse grid grid-cols-1 gap-4">
        <p class="h-6 w-64 bg-slate-200 rounded mx-auto"></p>
        <p class="h-6 w-32 bg-slate-200 rounded mx-auto"></p>
        <p class="h-6 w-32 bg-slate-200 rounded mx-auto"></p>
        <div class="h-32 w-32 bg-slate-200 rounded-full mx-auto"></div>
      </div>
      <div v-else class="grid grid-cols-1 gap-4">
        <p class="text-2xl text-center">{{ countryName }}, {{ countryCode }}</p>
        <p class="text-2xl text-center">{{ temperature }}&deg;C</p>
        <p class="text-2xl text-center">{{ weather?.main }}</p>
        <img class="w-32 mx-auto" :src="weather?.icon" alt="weather icon" />
      </div>
    </section>
  </main>
</template>
