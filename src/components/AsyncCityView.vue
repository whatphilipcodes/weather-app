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
                new Date(CurrentWeatherData.currentTime))
                }}
            </p>
            <p class="text-8xl mb-8">
                {{ Math.round(CurrentWeatherData.main.temp )}} &deg;
            </p>
            <p>Feels like
                {{ Math.round(CurrentWeatherData.main.feels_like) }} &deg;
            </p>
            <p class="capitalize">
                {{ CurrentWeatherData.weather[0].description }}
            </p>
            <img class="w-[150px] h-auto"
                :src="`http://openweathermap.org/img/wn/${CurrentWeatherData.weather[0].icon}@2x.png`" alt="" />
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- 5 day forecast -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-8 font-bold">Five Days Forecast</h2>
                <div class="flex gap-10 overflow-x-scroll">
                    <div v-for="dayData in ForecastData" :key="dayData.dt" class="flex flex-col gap-4 items-center">
                        <p class="whitespace-nowrap text-md">
                            {{
                            new Intl.DateTimeFormat('en', { /*dateStyle: "medium"*/ weekday:"long"}).format(
                            new Date(dayData.currentDay))
                            }}
                        </p>
                        <img class="w-auto h-[50px] object-cover"
                            :src="`http://openweathermap.org/img/wn/${dayData.weather[0].icon}@2x.png`" alt="" />
                        <p class="text-xl">
                            {{ Math.round(dayData.main.temp) }} &deg;
                        </p>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute } from 'vue-router';
import env from '@/env.js';
import { list } from 'postcss';

const route = useRoute();
const getCurrentWeatherData = async () => {
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

const getForecastData = async () => {
    try {

        const weatherData_3h5d = await axios.get(
            `https://api.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lng}&appid=${env.openweatherAPIKey}&units=metric`)

        // calculate current date & time
        const localOffset = new Date().getTimezoneOffset() * 60000;

        // store forecast
        var forecastData = new Array();

        weatherData_3h5d.data.list.forEach((day) => {
            if (day.dt_txt.includes("12:00:00")) {
                // calculate hourly offset (deprecated -> different api)
                const utc = day.dt * 1000 + localOffset;
                day.currentDay = utc + 1000 * weatherData_3h5d.data.city.timezone;

                // append only measurements for 12:00h
                forecastData.push(day);
                //console.log(forecastData.day);
            }
        });

        return forecastData;
    } catch (err) {
        console.log(err);
    }
};
const CurrentWeatherData = await getCurrentWeatherData();
const ForecastData = await getForecastData();
</script>