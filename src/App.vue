<template>
  
  <div class="appcontent scroll-smooth" :class="{ 'dark' : darkmode }">
    <a class="modebtn hover:bg-black hover:text-white dark:text-white dark:border-white dark:hover:bg-white dark:hover:text-black transition-colors" @click="darkMode" ref="btnModeRef" v-on:mouseenter="toggleTooltipMode()" v-on:mouseleave="toggleTooltipMode()" >
      <MoonIcon v-if="!darkmode" class="h-4 w-4 text-grey-400"/>
      <SunIcon v-else class="h-4 w-4 text-white-400"/>
      mode
    </a>
    <div ref="tooltipModeRef" :class="{'hidden': !tooltipMode, 'block': tooltipMode, 'bg-black' : !darkmode, 'bg-white' : darkmode}" class="tooltip bg-black block z-50 text-sm max-w-xs text-left break-words rounded-lg">
      <div class="p-2" :class="{'text-black' : darkmode, 'text-white' : !darkmode}">
        {{ darkmode ? 'Light' : 'Dark' }} mode
      </div>
    </div>

    <main class="p-6" :class="{ 'initial' : weathers.length === 0}">
      <LocationMarkerIcon v-if="weathers.length === 0" class="h-8 w-8 text-purple-500 inline-block"/>
      <ClockIcon v-if="weathers.length === 0" class="h-8 w-8 text-purple-500 inline-block"/>
      <svg v-if="weathers.length === 0" class="h-10 w-10 text-purple-500 inline-block"  width="10" height="10" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">  <path stroke="none" d="M0 0h24v24H0z"/>  <path d="M7 18a4.6 4.4 0 0 1 0 -9h0a5 4.5 0 0 1 11 2h1a3.5 3.5 0 0 1 0 7h-12" /></svg>
      <h1 v-if="weathers.length === 0" class="text-3xl font-semibold dark:text-white">Time & Weather</h1>
      <h4 v-if="weathers.length === 0" class="dark:text-white text-lg mb-2.5">Enter location to find out what's the date, time and weather there.</h4>
      <div class="search-box" :class="{'mb-3.5' : weathers.length != 0}">
        <input 
        v-model="query" 
        @keypress="fetchWeather"
        type="text" 
        class="search-bar w-full border-2 rounded-full p-1" 
        placeholder="Search location.."/>
        <p v-if="query != ''" class="text-red-600 italic">{{ message }}</p>
        <small v-if="weathers.length != 0"><button class="clearBtn dark:text-white hover:underline" @click="clearLocation()">
          Clear all</button></small>
      </div>
      <small v-if="weathers.length === 0" class="dark:text-white italic">Example : New York</small>
      <br>
      <div class="weathers grid gap-3" :class="[ weathers.length > 1 ? 'lg:grid-cols-3 sm:grid-cols-2' : 'place-content-center' ]">
        <div class="p-2" :style="weathers.length < 2 ? 'min-width: 350px' : ''"
          v-for="( weatherItem, index ) in weathers" :key="index" stagger="50">
          <div class="weather-wrap shadow rounded-lg text-center ">
            <button class="transition transform hover:-translate-y-1 removeBtn bg-gray-400 text-white rounded-full hover:bg-gray-600 dark:bg-indigo-900 dark:hover:bg-gray-900" 
              @click="deleteLocation(index)">
              <XIcon class="h-4 w-4 text-white"/>
            </button>
            <div class="location-box">
              <div class="location text-center text-3xl dark:text-white font-bold">
                <LocationMarkerIcon class="h-6 w-6 text-purple-400"/>
                {{ weatherItem.name }}, {{ weatherItem.country }}
              </div>
              <div class="date italic dark:text-white text-lg">
                {{ weatherItem.datezone }} <br> {{ weatherItem.timezone }}
              </div>
            </div>
            <div class="weather-box bg-slate-200 dark:bg-gray-400">
              <img :src="'https://openweathermap.org/img/w/'+weatherItem.icon+'.png'"/>
              <div class="weather-icon" >
                <div class="weather"> {{ weatherItem.weather }} </div>
              </div>
              <div class="temp text-4xl">{{ Math.round(weatherItem.temp) }} °C</div>
            </div>
          </div>
        </div>
      </div>

    </main>
  </div>

</template>

<script>
import './assets/index.css'
import { ClockIcon, LocationMarkerIcon,XIcon,MoonIcon,SunIcon } from '@heroicons/vue/outline'
import moment from 'moment-timezone'
import { createPopper } from "@popperjs/core"
export default {
  name: 'app',
  mounted() {  
    document.title = 'Vue Time & Weather app';  
  }, 
  data() {
    return {
      api_key: '',
      url_base: 'http://api.openweathermap.org/data/2.5/',
      darkmode: false,
      query: '',
      weathers: [],
      message: '',
      tooltipMode: false,
    }
  },
  methods :{
    fetchWeather ( e ) {
      if ( e.key == "Enter" ){
        fetch( `${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}` )
          .then( response => {
              return response.json();   
          }).then( this.locationResult )
          .catch(error => {
            this.message = 'Location not found';
            return error;
          });
      }
    },
    locationResult (result) {
      this.weather = result;
      let repeated = this.weathers.some(e => e.name === result.name);
      if(result.main != 'undefined' && !repeated){
        let timezone = result.timezone;
        let timezoneInMinutes = timezone / 60;
        this.weathers.push({ 
          name: result.name, 
          country: result.sys.country, 
          weather: result.weather[0].main,
          icon: result.weather[0].icon,
          temp: result.main.temp,
          timezone: moment().utcOffset(timezoneInMinutes).format("h.mm a"),
          datezone: moment().utcOffset(timezoneInMinutes).format('dddd, Do MMMM YYYY'),
        });
        this.query = '';
      }
      
    },
    deleteLocation (index){
      this.weathers.splice(index,1)
    },
    clearLocation (){
      this.weathers.splice(0)
    },
    darkMode(){
      this.darkmode = !this.darkmode;
    },
    toggleTooltipMode: function(){
      this.tooltipMode = !this.tooltipMode;
      if(this.tooltipMode){
        createPopper(this.$refs.btnModeRef, this.$refs.tooltipModeRef, {
          placement: "bottom"
        });
      }
    },
  },
  components: { ClockIcon,LocationMarkerIcon, XIcon, MoonIcon,SunIcon }
}
</script>
