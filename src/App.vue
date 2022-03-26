<template>
  <div class="map">
    <l-map
      v-model="zoom"
      v-model:zoom="zoom"
      :center="center"
      @move="log('move')"
      @update:center="updateCenter"
    >
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      ></l-tile-layer>

      <l-marker :lat-lng="[lat, lng]" draggable @ready="getForecast" @dragend="updateMarker($event.target.getLatLng())"></l-marker>


    </l-map>
  </div>
  <div class="data-panel">
    
    <h1>Spot Weather</h1>

    <div>{{ location }}</div>
    <div>{{ coordinates[0].toFixed(4) }}, {{ coordinates[1].toFixed(4) }}</div>
    <br>
    <div v-for="item in filteredTImeSeries" :key="item.time" class="summary">
      <div class="day">{{ formatDate(item.time) }}</div>
      <div class="temp">{{ Math.round(item.dayMaxScreenTemperature) }}&deg;</div>
    </div>

  </div>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LMarker
} from "@vue-leaflet/vue-leaflet";
import "leaflet/dist/leaflet.css";
import axios from 'axios';
import dayjs from 'dayjs';

export default {
  name: 'App',
    components: {
    LMap,
    LTileLayer,
    LMarker
  },
  data() {
    return {
      zoom: 7,
      center: [50.373061634625195, -4.130752086639405],
      lat: 50.373061634625195, 
      lng: -4.130752086639405,
      location: '',
      coordinates: [50.3730, -4.1307],
      timeSeries: []
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
          'X-IBM-Client-Id': '875b363a-9623-4a8a-9c68-d51802b5be65',
          'X-IBM-Client-Secret': 'yR2hM6aN2lI3jO3eS2nL2qB8kK6oP8yC8qN6cA3nB4cY5uU0qO'
        }
      }
      const forecast = await axios.get('https://api-metoffice.apiconnect.ibmcloud.com/metoffice/production/v0/forecasts/point/daily?excludeParameterMetadata=false&includeLocationName=true&latitude=' + this.lat + '&longitude=' + this.lng, config);
      console.log(forecast);
      this.location = forecast.data.features[0].properties.location.name;
      this.timeSeries = forecast.data.features[0].properties.timeSeries;
      this.coordinates = forecast.data.features[0].geometry.coordinates;
    },
    log(a) {
      console.log(a);
    },
    updateCenter(coords) {
      this.center = coords
    },
    updateMarker(coords) {
      this.lat = coords.lat;
      this.lng = coords.lng;
      this.getForecast;
    },
    formatDate(dateString) {
      return dayjs(dateString).format('ddd');
    }
  }
}
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css?family=Proza+Libre');
  
  body {
    margin: 0;
    font-family: 'Proza Libre', sans-serif;
  }
  .map {
    height: 100vh;
    width: 100vw;
    position: relative;
    z-index: 0;
  }
  .data-panel {
    background-color: white;
    position: absolute;
    top: 100px;
    left: 100px;
    z-index: 2;
    padding: 30px;
  }

  .summary{
    border-bottom: 1px solid #f3f3f3;
    padding: 10px 0;
    display: flex;
    justify-content: space-between;
    &:last-child {
      border: none;
    }
  }
</style>
