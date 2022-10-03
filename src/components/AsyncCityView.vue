<template>
    <div class="flex flex-col flex-1 items-center">
        <!-- banner -->
        <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
            <p>
                You are currently previewing this city, click the "+" icon to start tracking this city.
            </p>
        </div>
        <!-- weather overview -->
        <div class="flex flex-col items-center text-white py-12">
            <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
            <p class="text-sm mb-12">
                <!-- Updated to new convention, see: https://stackoverflow.com/questions/65021891/for-javascripts-tolocaledatestring-are-there-new-standards-to-replace-datestyl -->
                {{
                new Intl.DateTimeFormat('en', { dateStyle: "medium", timeStyle: "medium" }).format(
                new Date(weatherData.currentTime))
                }}
            </p>
            <p class="text-8xl mb-8">
                {{ Math.round(weatherData.main.temp )}} &deg;
            </p>
            <p>Feels like
                {{ Math.round(weatherData.main.feels_like) }} &deg;
            </p>
            <p class="capitalize">
                {{ weatherData.weather[0].description }}
            </p>
            <img class="w-[150px] h-auto"
                :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" alt="">
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- 5 day forecast -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4">Five Days Forecast</h2>
                <div class="flex gap-10 overflow-x-scroll"></div>
            </div>
        </div>

    </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute } from 'vue-router';
import env from '@/env.js';

const route = useRoute();
const getWeatherData = async () => {
    try {
        //original: `https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=${env.openweatherAPIKey}&units=imperial`
        // using different endpoint for pricing reasons

        const weatherData_current = await axios.get(
            `https://api.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lng}&appid=${env.openweatherAPIKey}&units=metric`)

        // const weatherData_3h5d = await axios.get(
        //     `https://api.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lng}&appid=${env.openweatherAPIKey}&units=metric`)

        // Due to the usage of different endpoints, different props are used!
        // calculate current date & time
        const localOffset = new Date().getTimezoneOffset() * 60000;
        const utc = weatherData_current.data.dt * 1000 + localOffset;
        weatherData_current.data.currentTime = utc + 1000 * weatherData_current.data.timezone;

        // calculate hourly offset (deprecated -> different api)
        // weatherData_current.data.hourly.forEach((hour) => {
        //     const utc = hour.dt * 1000 + localOffset;
        //     hour.currentTime = utc + 1000 * weatherData_current.data.timezone_offset;
        // });

        return weatherData_current.data;
    } catch (err) {
        console.log(err);
    }
};
const weatherData = await getWeatherData();
console.log(weatherData);
</script>