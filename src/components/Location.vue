<script setup lang="ts">
import {onMounted, ref, watch} from 'vue';

const distance = ref(1000);
const distanceDisplay = ref("");
const lat = ref(0);
const long = ref(0);
const withinBoundingBox = ref(false);

const LATITUDE = 44.978977;
const LONGITUDE = -93.256022;
const RADIUS_OF_EARTH = 6371; // kilometers
const BOUNDING_BOX_FEET = 15;

function error() {

}

function success(position: GeolocationPosition) {
  lat.value = position.coords.latitude;
  long.value = position.coords.longitude;

  calculateDistance();
}

function calculateDistance() {
  const dLat = deg2rad(LATITUDE-lat.value);
  const dLong = deg2rad(LONGITUDE-long.value);  

  const lat1 = deg2rad(lat.value);
  const lat2 = deg2rad(LATITUDE);

  const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
    Math.sin(dLong/2) * Math.sin(dLong/2) * Math.cos(lat1) * Math.cos(lat2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  const dKm = RADIUS_OF_EARTH * c;
  const dMi = dKm * 0.621371;

  distance.value = dMi;
  let dFeet: number = Infinity;
  if (dMi > 1) {
    distanceDisplay.value = dMi.toFixed(3) + " mi";
  } else {
    dFeet = dMi * 5280;
    distanceDisplay.value = dFeet.toFixed(0) + " feet";

    if (dFeet <= BOUNDING_BOX_FEET) {
      withinBoundingBox.value = true;
    } else {
      withinBoundingBox.value = false;
    }
  }
  console.log({
    lat: lat.value,
    long: long.value,
    distance: dFeet,
    withinBoundingBox: withinBoundingBox.value
  })
}

function deg2rad(deg: number) {
  return deg * (Math.PI/180);
}


onMounted(() => {
  navigator.geolocation.watchPosition(success, error);
});



</script>

<template>
  <div class="location" v-if="!withinBoundingBox">
    <div class="coords">
      Your Latitude: {{ lat }}
    </div>
    <div class="coords">
      Your Longitude: {{  long }}
    </div>
    <div class="distance">
      Distance to Clue: {{ distanceDisplay }}
    </div>
  </div>
  <div class="location" v-if="withinBoundingBox">
    You made it!
  </div>
</template>


<style scoped>
  .coords {
    font-size: 2rem;
  }

  .distance {
    margin-top: 2rem;
    font-weight: bold;
    font-size: 2rem;
  }
</style>