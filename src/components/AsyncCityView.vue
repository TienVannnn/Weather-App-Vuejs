<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the "+" icon to start
        tracking this city.
      </p>
      <button @click="changeUnit">Change unit</button>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(localTime).toLocaleDateString("en-us", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          new Date(localTime).toLocaleTimeString("en-us", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="text-8xl mb-8">{{ Math.round(currentData.main.temp) }}&deg;</p>
      <p>
        Feels like
        {{ Math.round(currentData.main.feels_like) }} &deg;
      </p>
      <p class="capitalize">
        {{ currentData.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${currentData.weather[0].icon}@2x.png`"
        alt=""
      />
    </div>
    <hr class="border-white border-opacity-10 border w-full" />
    <!-- widgets weather -->
    <div id="openweathermap-widget-1" class="mt-8"></div>
    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
      v-if="route.query.id"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { watch } from "vue";

const unit = ref("metric");
const changeUnit = () => {
  unit.value = unit.value === "metric" ? "imperial" : "metric";
};

const route = useRoute();
const localTime = ref(null);

const getWeatherData = async () => {
  try {
    const res = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lng}&appid=da66ac3e21a7ff2d9259184a0a8f3b2e&units=${unit.value}`
    );
    const weatherData = res.data;

    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.dt * 1000 + localOffset;
    localTime.value = new Date(utc + weatherData.timezone * 1000);
    return weatherData;
  } catch (err) {
    console.error(err);
  }
};
const currentData = await getWeatherData();
// widgets
onMounted(() => {
  if (currentData) {
    const d3Script = document.createElement("script");
    d3Script.src = "https://d3js.org/d3.v5.min.js";
    d3Script.async = true;
    document.head.appendChild(d3Script);
    window.myWidgetParam ? window.myWidgetParam : (window.myWidgetParam = []);
    window.myWidgetParam.push({
      id: 1,
      cityid: currentData.id, // ID thành phố từ API
      appid: "da66ac3e21a7ff2d9259184a0a8f3b2e",
      units: unit.value,
      containerid: "openweathermap-widget-1",
    });
    const script = document.createElement("script");
    script.async = true;
    script.charset = "utf-8";
    script.src =
      "//openweathermap.org/themes/openweathermap/assets/vendor/owm/js/weather-widget-generator.js";
    document.body.appendChild(script);
  } else {
    console.error("Lỗi.");
  }
});
// remove city from localStorage
const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter((city) => city.id !== route.query.id);
  localStorage.setItem("savedCities", JSON.stringify(updatedCities));
  router.push({
    name: "home",
  });
};
</script>

<style lang="scss" scoped></style>
