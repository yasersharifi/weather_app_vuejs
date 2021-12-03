<template>
    <v-app id="app">
        <v-container class="mt-8">
            <v-row class="mb-5">
                <v-col cols="12">
                    <h1 class="white--text" style="font-size: 90px">Simple Weather App</h1>
                </v-col>
            </v-row>
            <v-row>
                <v-col cols="12">
                    <v-autocomplete v-model="city" :items="items" clearable color="white" dark label="Search By Ciry"></v-autocomplete>
                </v-col>
            </v-row>
            <v-row>
                <v-col cols="12">
                    <v-btn dark color="red" class="text-center" @click="searchCityWeather" x-large>
                        Search
                        <v-icon class="ms-2" small>mdi-arrow-right</v-icon>
                    </v-btn>
                </v-col>
            </v-row>
            <v-row class="mt-6">
                <v-col cols="6">
                    <v-card class="pa-8 cartArea">
                        <v-card-title>
                            {{ cityWeatherInfo.name }}
                            <v-badge :content="cityWeatherInfo.country" color="orange" class="ml-2"></v-badge>
                        </v-card-title>
                        <v-card-actions>
                            <h1 style="font-size: 70px">
                                <strong>{{ cityWeatherInfo.degree }}</strong>
                                <sup>°c</sup>
                            </h1>
                        </v-card-actions>
                        <v-card-text>
                            <v-img :src="cityWeatherInfo.icon" width="150"/>
                        </v-card-text>
                        <v-card-text>
                            <p class="body-1"><strong>{{ cityWeatherInfo.weather }}</strong></p>
                        </v-card-text>
                    </v-card>
                </v-col>

                <v-col cols="6">
                    <v-card class="pa-8 cartArea">
                        <v-badge color="orange" content="default" style="position: absolute;top: 25px;right: 52px;"></v-badge>
                        <v-card-title>
                            {{ defaultCityWeatherInfo.name }}
                            <v-badge :content="defaultCityWeatherInfo.country" color="orange" class="ml-2"></v-badge>
                        </v-card-title>
                        <v-card-actions>
                            <h1 style="font-size: 70px">
                                <strong>{{ defaultCityWeatherInfo.degree }}</strong>
                                <sup>°c</sup>
                            </h1>
                        </v-card-actions>
                        <v-card-text>
                            <v-img :src="defaultCityWeatherInfo.icon" width="150"/>
                        </v-card-text>
                        <v-card-text>
                            <p class="body-1"><strong>{{ defaultCityWeatherInfo.weather }}</strong></p>
                        </v-card-text>
                    </v-card>
                </v-col>
            </v-row>
        </v-container>
    </v-app>
</template>

<script>
import axios from "axios";

export default {
    name: 'App',
    data: function () {
        return {
            items: [],
            city: 'Fars',
            cityWeatherInfo: {},
            defaultCity: "Tehran",
            defaultCityWeatherInfo: {},
        }
    },
    components: {},
    methods: {
        searchCityWeather: function () {
            let lat = null;
            let lon = null;
            let self = this;
            console.log(this.city)
            const findLatAndLon = new Promise((resolve) => {
                // find lat and lon of cities
                axios
                    .get("./data/iranCities.json")
                    .then((response) => {
                        response.data.forEach((value) => {
                            if (value.slug === self.city) {
                                lat = value.latitude;
                                lon = value.longitude;
                                resolve({lat: lat, lon: lon});
                            }
                        });
                    })
            }) // end of promise

            findLatAndLon.then((response) => {
                lat = response.lat;
                lon = response.lon;
                let url = `http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=9a2be281fe85661eb2b845c7111518a3`;
                axios
                    .get(url)
                    .then((response) => {
                        this.cityWeatherInfo = {
                            name: response.data.name,
                            country: response.data.sys.country,
                            weather: response.data.weather[0].description || response.data.weather[0].main,
                            icon: `http://openweathermap.org/img/wn/${response.data.weather[0].icon}@2x.png`,
                            degree: response.data.wind.deg
                        }
                    })
            })// end of findLatAndLon
        },
        defaultCityWeather: function () {
            let url = `http://api.openweathermap.org/data/2.5/weather?q=${this.defaultCity}&appid=9a2be281fe85661eb2b845c7111518a3`;
            axios
                .get(url)
                .then((response) => {
                    this.defaultCityWeatherInfo = {
                        name: response.data.name,
                        country: response.data.sys.country,
                        weather: response.data.weather[0].description || response.data.weather[0].main,
                        icon: `http://openweathermap.org/img/wn/${response.data.weather[0].icon}@2x.png`,
                        degree: response.data.wind.deg
                    }
                })
        }
    },
    mounted() {
        let self = this;
        axios
            .get("./data/iranCities.json")
            .then((response) => {
                response.data.forEach((value) => {
                    self.items.push(value.slug);
                });
            })

        // call searchCityWeather methods
        this.searchCityWeather();

        // call defaultCityWeather methods
        this.defaultCityWeather();

    }

}
</script>

<style>
#app {
    width: 100%;
    min-height: 100vh;
    height: 100%;
    position: relative;
    background: rgb(2,0,36);
    background: linear-gradient(90deg, rgba(2,0,36,1) 0%, rgba(9,9,121,1) 25%, rgba(5,137,163,1) 100%);
}
.cartArea {
    box-shadow: 0 12px 20px 22px #e5e1ff45 !important;
}
</style>
