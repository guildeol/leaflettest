<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue'
import axios from 'axios'

import "leaflet/dist/leaflet.css";
import leaflet from 'leaflet';
import { LatLng } from 'leaflet';

export default defineComponent({
  name: 'MyMap',

  data() {
      return {
        address: '' as String,
        requested: false as boolean,
        map: null,
      }
  },

  setup() {
    onMounted(() => {
      map = leaflet.map('map').setView([51.505, -0.09], 13);

      leaflet.tileLayer(
        'https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
          maxZoom: 19,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(map);
    });
  },

  methods: {
    getCoordinates(response) : LatLng {
      let coords = response.data.features[0].geometry.coordinates;

      return new leaflet.LatLng(coords[1], coords[0]);
    },

    requestForwardGeocoding(address: string) {
      const apiKey = '';
      let encodedAddress = encodeURIComponent(address);

      if (apiKey == '')
        return console.error("Add your own geoapify API key!");

      this.requested = true;

      if (address == '')
        return;

      let url = 'https://api.geoapify.com/v1/geocode/search?text=' + encodedAddress + '&apiKey=' + apiKey;

      axios.get(url).then((response) => {
        const newZoomLevel: number = 50;
        let latLon: LatLng = this.getCoordinates(response) 

        map.setView(latLon, newZoomLevel);

        let marker = new leaflet.Marker(latLon);
        marker.addTo(map);
      });
    }
  }
});

</script>

<template>
  <div style="padding-bottom: 15px;">
    <p>Enter the desired address</p>
    <input v-model="address" placeholder="Type here..."/>
    <button @click="requestForwardGeocoding(address)">Send</button>
  </div>

  <div v-if="requested">
    <p v-if="address == ''" style="color: red;">Please type in an address!</p>
    <p v-else> Coordinates for {{ address }} </p>
  </div>

  <div id="map" style="height: 500px; width: 700px;"></div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
