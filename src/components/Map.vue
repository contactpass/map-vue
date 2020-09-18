<template>
  <div>
    <!--<h1>Your location</h1>
    <p>{{ myCoordinates.lat }} Latitude, {{ myCoordinates.lng }} Longtitude</p>
    <h1>Map location</h1>
    <p>{{ mapCoordinates.lat }} Latitude, {{ mapCoordinates.lng }} Longtitude</p> -->
    <GmapMap 
        :center="myCoordinates" 
        :zoom="7" 
        style="width: 100%; height:100%; position: inherit !important;"
        ref="mapRef">
      <GmapMarker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :clickable="true"
        :draggable="true"
        @click="center=m.position"
      />
    </GmapMap>
  </div>
</template>

<script>
export default {
  data() {
    return {
      map: null,
      myCoordinates: {
        lat: 0,
        lng: 0,
      }
    };
  },
  created() {
    this.$getLocation({}).then((coordinates) => {
      this.myCoordinates = coordinates;
    })
    .catch(error => alert(error))
  },
  mounted() {
    this.$refs.mapRef.$mapPromise.then(map => this.map = map)
  },
  computed: {
    mapCoordinates() {
      if(!this.map) {
        return {
          lat: 0,
          lng: 0,
        }
      }

      return {
        lat: this.map.getCenter().lat().toFixed(4),
        lng: this.map.getCenter().lng().toFixed(4),
      }
      
    }
  }
};
</script>

