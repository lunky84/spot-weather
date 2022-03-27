<template>
    <div class="map">
        <l-map
        v-model="zoom"
        v-model:zoom="zoom"
        :center="center"
        @update:center="updateCenter"
        >
          <l-tile-layer url="https://tiles.stadiamaps.com/tiles/alidade_smooth/{z}/{x}/{y}{r}.png"></l-tile-layer>
          <l-marker :lat-lng="[lat, lng]" draggable @ready="emitGetForecast" @dragend="emitMarkerChange($event.target.getLatLng())"></l-marker>
        </l-map>
    </div>
</template>
<script>
import {
  LMap,
  LTileLayer,
  LMarker
} from "@vue-leaflet/vue-leaflet";
import "leaflet/dist/leaflet.css";
export default {
  name: "LeafletMap",
  components: {
    LMap,
    LTileLayer,
    LMarker
  },
  props: {
    lat: Number,
    lng: Number
  },
  data() {
    return {
      zoom: 9,
      center: [50.373061634625195, -4.130752086639405],
    }
  },
  methods: {
    updateCenter(coords) {
      this.center = coords
    },
    emitGetForecast() {
     this.$emit('onReady')
    },
    emitMarkerChange(latLng) {
     this.$emit('markerChange', latLng)
    }
  }
};
</script>