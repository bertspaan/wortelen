<template>
  <div id="app">
    <main>
      <div class="map-container">
        <div id="map" ref="map"/>
      </div>
    </main>
  </div>
</template>

<script>

import { lineChunk, pointToLineDistance } from '@turf/turf'

export default {
  name: 'app',
  components: {

  },
  data () {
    return {
      map: undefined,
      loaded: false,
      animating: false,
      glAdded: false,
      currentDistance: 0,
      maxDistance: 1000
    }
  },
  methods: {
    getLineWidth: function (maxDistance) {
      const maxWidth = 15
      return ['interpolate',
        ['linear'],
        ['get', 'distance'],
        0, maxWidth,
        Math.max(maxDistance - 50, 1), maxWidth,
        Math.max(maxDistance, 2), 0
      ]
    },
    reset: function () {
      this.currentDistance = 0
      this.updateLayerStyle()
    },
    updateLayerStyle: function () {
      const lineWidth = this.getLineWidth(this.currentDistance)
      this.map.setPaintProperty('chunks', 'line-width', lineWidth)
    },
    startAnimating: function () {
      this.reset()
      this.animating = true

      const vm = this

      function animate () {
        vm.currentDistance += 5
        vm.updateLayerStyle()

        if (vm.animating && vm.currentDistance < vm.maxDistance) {
          requestAnimationFrame(animate)
        }
      }

      animate()
    },
    stopAnimating: function () {
      this.animating = false
    }
  },
  mounted: function () {
    // eslint-disable-next-line
    const map = new mapboxgl.Map({
      container: 'map',
      style: 'https://ta.webmapper.nl/wm/styles/topography.json',
      center: [4.922, 52.369],
      zoom: 9,
      minZoom: 15,
      hash: true
    })

    map.on('load', () => {
      this.loaded = true

      const chunks = {
        type: 'FeatureCollection',
        features: []
      }

      map.addSource('chunks', {
        type: 'geojson',
        data: chunks
      })

      map.addLayer({
        id: 'chunks',
        type: 'line',
        source: 'chunks',
        paint: {
          'line-color': '#022520',
          'line-opacity': 0.8,
          'line-width': ['interpolate',
            ['linear'],
            ['get', 'distance'],
            0, 6,
            this.maxDistance, 0
          ]
        }
      })
    })

    map.on('mousedown', (event) => {
      if (!this.loaded) {
        return
      }

      const point = {
        type: 'Point',
        coordinates: [event.lngLat.lng, event.lngLat.lat]
      }

      const features = map.querySourceFeatures('wm_visdata', {
        sourceLayer: ['roads']
      })

      const chunks = []
      let id = 0

      features.forEach((feature) => {
        const featureChunks = lineChunk(feature.geometry, 25, {units: 'meters'})
        featureChunks.features.forEach((feature) => {
          chunks.push({
            type: 'Feature',
            id: id++,
            properties: {
              distance: pointToLineDistance(point, feature, {units: 'meters'})
            },
            geometry: feature.geometry
          })
        })
      })

      const geojson = {
        type: 'FeatureCollection',
        features: chunks
      }


      if (!this.glAdded) {
        map.addLayer({
          id: 'custom',
          type: 'custom',
          renderingMode: '3d',

          onAdd: function (map, mbxContext) {
            this.threebox = new Threebox(
              map,
              mbxContext,
              {defaultLights: true}
            )

            chunks.forEach((chunk, chunkIndex) => {
              // chunk.properties.distance
              const lineOptions = {
                geometry: chunk.geometry.coordinates.map((coordinate, index) =>
                  [
                    ...coordinate,
                    index === 0 ? (chunkIndex === 0 ? 0 : chunks[chunkIndex - 1].properties.distance / 5) : chunk.properties.distance / 5
                  ]
                ),
              	color: 0x33ee22 * chunk.properties.distance,
              	width: 6
              }

              const lineMesh = this.threebox.line(lineOptions)
              this.threebox.add(lineMesh)
            })
          },
          render: function (gl, matrix) {
            this.threebox.update()
          }
        })
      }


      this.geojson = geojson
      map.getSource('chunks').setData(geojson)
      this.startAnimating()
    })

    map.on('mouseup', () => {
      if (!this.loaded) {
        return
      }

      this.stopAnimating()
    })

    this.map = map
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  position: absolute;
  top: 0;
  width: 100%;
  height: 100%;
}

#app {
  display: flex;
  flex-direction: column;

  width: 100%;
  height: 100%;
  /* max-width: 1000px; */
  margin: 0 auto;
}

main {
  flex-grow: 1;
  position: relative;
}

#map {
  width: 100%;
  height: 100%;
}

.map-container {
  top: 0;
  position: absolute;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}
</style>
