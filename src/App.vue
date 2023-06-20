<template>
  <div id="app">
    <input type="text" v-model="input" @input="onInput" placeholder="Type a state or territory">
    <div v-for="suggestion in suggestions" :key="suggestion" @click="selectSuggestion(suggestion)">
      {{ suggestion }}
    </div>
    <div id="map"></div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      input: '',
      suggestions: [],
      map: null,

    };
  },
  mounted() {
    window.initMap = this.initMap;
  },
  methods: {
    onInput() {
      axios.post('http://localhost:4000/graphql', {
        query: `{ getSuggestions(input: "${this.input}") }`
      }).then(response => {
        this.suggestions = response.data.data.getSuggestions;
      }).catch(error => {
        console.log(error);
      });
    },
    selectSuggestion(suggestion) {
      this.input = suggestion;
      this.suggestions = [];
      this.highlightState(suggestion);
    },
    initMap() {
      this.map = new window.google.maps.Map(document.getElementById('map'), {
        center: { lat: 37.0902, lng: -95.7129 },
        zoom: 5,
      });
    },
    
    highlightState(state) {
      if (this.currentOverlay) {
        this.currentOverlay.setMap(null);
      }


      fetch('us-states.json')
        .then(response => response.json())
        .then(data => {
          
          const stateFeature = data.features.find(feature => feature.properties.name === state);
          if (stateFeature) {
            this.currentOverlay = new window.google.maps.Data();
            this.currentOverlay.addGeoJson(stateFeature);
            this.currentOverlay.setStyle({
              fillColor: 'red',
              strokeWeight: 1
            });
            this.currentOverlay.setMap(this.map);
            
            
            const bounds = new window.google.maps.LatLngBounds();
            this.currentOverlay.forEach(feature => {
              feature.getGeometry().forEachLatLng(latlng => {
                bounds.extend(latlng);
              });
            });
            this.map.fitBounds(bounds);
          }
        });
    }
  },
};
</script>




<style>
#map {
  height: 500px;
  width: 100%;
}
</style>
