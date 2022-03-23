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

      <l-marker :lat-lng="[50, 50]" draggable @update:lat-lng="updateMarker" @moveend="getForecast"></l-marker>


    </l-map>
  </div>
  <div class="data-panel">
    
    <h1>Spot Weather</h1>

    <div>{{ location }}</div>
    <ul id="example-1">
      <li v-for="item in filteredTImeSeries" :key="item.time">
        {{ item.time }}
      </li>
    </ul>

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
export default {
  name: 'App',
    components: {
    LMap,
    LTileLayer,
    LMarker
  },
  data() {
    return {
      zoom: 3,
      center: [47.41322, -1.219482],
      lat: 47.41322, 
      lng: -1.219482,
      location: '',
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
      this.location = forecast.data.features[0].properties.location.name
      this.timeSeries = forecast.data.features[0].properties.timeSeries
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
    },
  }
}
</script>

<style>
  body {
    margin: 0;
  }
  .map {
    height: 100vh;
    width: 100vw;
    position: relative;
    z-index: 0;
  }
  .data-panel {
    background-color: bisque;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 2;
  }
</style>
