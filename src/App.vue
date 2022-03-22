<template>
  <div id="app">
    
    <h1>Spot Weather</h1>

    <button @click="getForecast()">Get Forecast</button>

    <div>{{ location }}</div>

  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'App',
  data() {
    return {
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
    }
  }
}
</script>

<style>

</style>
