<template>
<div style="height: 100vh; width: 100vw;">
    <l-map
      v-model="zoom"
      v-model:zoom="zoom"
      :center="[47.41322, -1.219482]"
      @move="log('move')"
    >
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      ></l-tile-layer>

      <l-marker :lat-lng="[50, 50]" draggable @moveend="log('moveend')">
        <l-popup>
          lol
        </l-popup>
      </l-marker>


    </l-map>
  </div>
  <div id="app">
    
    <h1>Spot Weather</h1>

    <button @click="getForecast()">Get Forecast</button>

    <div>{{ location }}</div>

  </div>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LMarker,
  LPopup,
} from "@vue-leaflet/vue-leaflet";
import "leaflet/dist/leaflet.css";
import axios from 'axios';
export default {
  name: 'App',
    components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup
  },
  data() {
    return {
      zoom: 2,
      location: ''
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
      const forecast = await axios.get('https://api-metoffice.apiconnect.ibmcloud.com/metoffice/production/v0/forecasts/point/daily?excludeParameterMetadata=false&includeLocationName=true&latitude=50.388139&longitude=-4.137786', config);
      console.log(forecast);
      this.location = forecast.data.features[0].properties.location.name
    },
    log(a) {
      console.log(a);
    },
  }
}
</script>

<style>

</style>
