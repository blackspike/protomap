<template lang="pug">
.map-wrap
  .map(ref="mapContainer")
</template>

<script setup>
import { ref, onMounted } from 'vue'
import slugify from 'slugify'
import pubs from '../data/pubs.json'
import maplibregl from 'maplibre-gl'
import layers from 'protomaps-themes-base'
import * as pmtiles from "pmtiles"

let protocol = new pmtiles.Protocol()
maplibregl.addProtocol("pmtiles", protocol.tile)

const mapContainer = ref(null)
const map = ref(null)

onMounted(() => {

  const initialState = { lat: 50.835351, lng: -0.147776, zoom: 13 }

  map.value = new maplibregl.Map({
    container: mapContainer.value,
    center: [initialState.lng, initialState.lat],
    zoom: initialState.zoom,
    style: {
      version: 8,
      glyphs: 'https://cdn.protomaps.com/fonts/pbf/{fontstack}/{range}.pbf',
      sources: {
        "protomaps": {
          type: "vector",
          url: "pmtiles://https://protomaps-pubs.netlify.app/brighton-hove.pmtiles",
          attribution: '<a href="https://protomaps.com">Protomaps</a> © <a href="https://openstreetmap.org">OpenStreetMap</a>'
        }
      },
      layers: layers("protomaps", "light")
    }
  })

  map.value.addControl(new maplibregl.NavigationControl(), 'top-right')

  pubs.map(pub => {

    const popDiv = document.createElement('div')
    const popLink = document.createElement('a')
    const popTitle = document.createElement('h3')
    popTitle.innerText = `${pub.name}, ${pub.address_1}`
    popLink.href = `https://www.brighton.dog/drink/${pub.id}/${slugify(pub.name, { strict: true, lower: true })}`
    popLink.innerText = 'link'
    popDiv.append(popTitle, popLink)
    const popup = new maplibregl.Popup({ offset: 25 }).setDOMContent(popDiv)

    const el = document.createElement('div')
    el.className = 'marker'
    el.innerText = '🍺'
    el.style.fontSize = '24px'
    el.style.width = '24px'
    el.style.height = '24px'

    new maplibregl.Marker(el)
      .setLngLat([pub.lng, pub.lat])
      .setPopup(popup)
      .addTo(map.value)
  })

})

</script>

<style lang="scss" scoped>
.map-wrap {
  outline: 2px dotted orangered;
  position: relative;
  width: 100%;
  height: 100vh;
  /* calculate height of the screen minus the heading */
}

.map {
  position: absolute;
  width: 100%;
  height: 100%;
}
</style>

<style>
/* .maplibregl-ctrl-attrib {
  position: absolute;
  bottom: 0;
  padding: 1rem;
  font-size: .75rem;
} */

.mapboxgl-popup-content {
  color: #222;
}

.marker {
  display: block;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  padding: 0;
}

h3 {
  margin: 0;
}
</style>