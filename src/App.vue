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
        url="https://tiles.stadiamaps.com/tiles/alidade_smooth/{z}/{x}/{y}{r}.png"
      ></l-tile-layer>

      <l-marker :lat-lng="[lat, lng]" draggable @ready="getForecast" @dragend="updateMarker($event.target.getLatLng())"></l-marker>


    </l-map>
  </div>
  <div class="data-panel">
    
    <div class="overview">
      <div class="location">{{ location }}</div>
      <div>{{ dateTime }}</div>
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
      zoom: 9,
      center: [50.373061634625195, -4.130752086639405],
      lat: 50.373061634625195, 
      lng: -4.130752086639405,
      location: '',
      dateTime: dayjs().format('ddd, D MMMM HH:mm'),
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
      this.getForecast();
    },
    formatDate(dateString) {
      return dayjs(dateString).format('dddd');
    }
  }
}
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@100;300&display=swap');
  
  body {
    margin: 0;
    font-family: 'Montserrat', sans-serif;
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
    
    border-radius: 15px;
    font-size: 13px;
    background: rgb(255,134,42);
    background: linear-gradient(0deg, rgba(255,134,42,1) 0%, rgba(255,203,42,1) 100%);
    color: white;
  }

  .overview {
    padding: 24px 30px;
    .location {
      font-size: 20px;
      margin: 0 0 15px 0;
    }
  }

  .outlook {
    border-top: 1px solid rgba(255, 255, 255, 0.4);
    padding: 24px 30px;
  }

  .summary {
    padding: 6px 0;
    min-width: 220px;
    display: flex;
    justify-content: space-between;
    &:last-child {
      border: none;
    }
    .day {
      flex: 0 0 90px;
    }
  }
</style>
