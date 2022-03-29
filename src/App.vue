<template>
  <LeafletMap :lat="lat" :lng="lng" @onReady="getForecast" @markerChange="updateMarker" />
  <div class="data-panel" :class="climate" v-if="filteredTImeSeries.length">
    
    <div class="overview">
      <div class="location">{{ location }}</div>
      <div class="date-time">{{ dateTime }}</div>
      <div>{{ coordinates[0].toFixed(4) }}, {{ coordinates[1].toFixed(4) }}</div>
    </div>
    <div class="outlook">
      <div v-for="item in filteredTImeSeries" :key="item.time" class="summary">
        <div class="day">{{ formatDate(item.time) }}</div>
        <div class="rain">{{ item.dayProbabilityOfRain }}%</div>
        <div class="temp">{{ Math.round(item.dayMaxScreenTemperature) }}&deg;</div>
      </div>
    </div>

  </div>
</template>

<script>
import LeafletMap from "./components/LeafletMap.vue";
import axios from 'axios';
import dayjs from 'dayjs';
import { X_IBM_CLIENT_ID, X_IBM_CLIENT_SECRET } from './../apikey';



export default {
  name: 'App',
  components: {
      LeafletMap,
  },
  data() {
    return {
      lat: 50.373061634625195, 
      lng: -4.130752086639405,
      location: '',
      dateTime: dayjs().format('ddd, D MMMM HH:mm'),
      coordinates: [50.3730, -4.1307],
      timeSeries: [],
      climate: ''
    }
  },
  computed: {
    filteredTImeSeries() {
      // Remove the first entry as that is yesterday
      return this.timeSeries.slice(1);  
    }
  },
  methods: {
    async getForecast() {
      let config = {
        headers: {
          'Accept': 'application/json',
          'X-IBM-Client-Id': X_IBM_CLIENT_ID,
          'X-IBM-Client-Secret': X_IBM_CLIENT_SECRET
        }
      }
      const forecast = await axios.get('https://api-metoffice.apiconnect.ibmcloud.com/metoffice/production/v0/forecasts/point/daily?excludeParameterMetadata=false&includeLocationName=true&latitude=' + this.lat + '&longitude=' + this.lng, config);
      // console.log(forecast);
      this.location = forecast.data.features[0].properties.location.name;
      this.timeSeries = forecast.data.features[0].properties.timeSeries;
      this.coordinates = forecast.data.features[0].geometry.coordinates;
      this.deriveClimate();
    },
    updateMarker(coords) {
      this.lat = coords.lat;
      this.lng = coords.lng;
      this.getForecast();
    },
    formatDate(dateString) {
      return dayjs(dateString).format('dddd');
    },
    deriveClimate() {
      let temp = this.timeSeries[1].dayMaxScreenTemperature;
      switch(true) {
        case temp < 8:
          this.climate = 'cold';
          break;
        case temp >= 8 && temp < 20:
          this.climate = 'mild';
          break;
        case temp >= 20:
          this.climate = 'hot';
          break;
      }      
    }
  }
}
</script>
