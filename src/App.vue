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

export default {
  name: 'app',
  components: {

  },
  data () {
    return {
      map: undefined
    }
  },
  mounted: function () {
    // eslint-disable-next-line
    const map = new mapboxgl.Map({
      container: 'map',
      style: 'https://ta.webmapper.nl/wm/styles/topography.json',
      center: [4.922, 52.369],
      zoom: 9,
      hash: true
    })

    map.on('load', () => {

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
