<template>
  <section class="">
    <form @submit.prevent="search">
      <input v-model="ubigeo" pattern="^\d{6}$" required>
    </form>
    <div ref="map" class="map"></div>
    <div v-if="district" class="">
      {{ district.properties.c_distrito }}
    </div>
  </section>
</template>

<script>
/* global google */

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
  width: 600px;
  height: 400px;
}
</style>
