<template>
  <section>
    <div class="">
      <div class="form">
        <form @submit.prevent="search">
          <label for="code"><strong>Código Ubigeo: </strong></label>
          <input v-model="ubigeo" id="code" pattern="^\d{6}$" required>
        </form>
      </div>
      <div class="container">
        <div ref="map" class="map"></div>
        <div v-if="district" class="">
          {{ district.properties.c_distrito }}
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  data () {
    return {
      ubigeo: '',
      district: null,
      map: null,
      polygon: null
    }
  },
  mounted () {
    this.initMap()
  },
  watch: {
    district: {
      deep: true,
      handler: 'refreshMap'
    }
  },
  methods: {
    async search () {
      try {
        this.district = await this.$axios.$get('/ubigeos/' + this.ubigeo)
      } catch (e) {
        this.district = null
      }
    },
    initMap () {
      this.map = new google.maps.Map(this.$refs.map, {
        zoom: 14,
        center: {lat: 24.886, lng: -70.268},
        mapTypeId: 'terrain'
      })
    },
    refreshMap () {
      this.polygon && this.polygon.setMap(null)
      if (!this.district) return
      const paths = this.district.geometry.coordinates[0].map(ll => ({ lat: ll[1], lng: ll[0] }))
      this.polygon = new google.maps.Polygon({
        paths,
        strokeColor: '#FF0000',
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: '#FF0000',
        fillOpacity: 0.35
      })
      this.polygon.setMap(this.map)
      let x1 = Math.min(...paths.map(p => p.lat))
      let y1 = Math.min(...paths.map(p => p.lng))
      let x2 = Math.max(...paths.map(p => p.lat))
      let y2 = Math.max(...paths.map(p => p.lng))
      let center = {
        lat: x1 + ((x2 - x1) / 2),
        lng: y1 + ((y2 - y1) / 2)
      }
      this.map.setCenter(new google.maps.LatLng(center.lat, center.lng))
    }
  }
}
</script>

<style>
.map {
  width: 800px;
  margin: 0 auto;
  height: 400px;
}
@media (max-width: 800px) {
  .map {
    width: 100%;
  }
}
.pdq-logo {
  width: 100%;
  background-color: black;
  text-align: center;
  padding: 1rem 0;
}
.form {
  margin: 2rem 0;
  text-align: center;
}
.container {
  width: 100%;
  margin: 0 auto;
}
</style>
