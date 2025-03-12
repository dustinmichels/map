<script setup lang="ts">
import { onMounted, ref, onBeforeUnmount } from 'vue'
// Need to import using the following syntax for Vue 3 with globe.gl
// Note: We're using dynamic import to avoid SSR issues
import { nextTick } from 'vue'

interface Properties {
  ADMIN: string
  ISO_A2: string
  POP_EST: number
  [key: string]: any
}

interface Feature {
  type: string
  properties: Properties
  geometry: any
}

interface GeoJSON {
  type: string
  features: Feature[]
}

const globeContainer = ref<HTMLElement | null>(null)
let globeInstance: any = null

// Create the resize handler outside the onMounted hook
const handleResize = () => {
  if (globeInstance) {
    globeInstance.width(window.innerWidth)
    globeInstance.height(window.innerHeight)
  }
}

// Register the cleanup function before any await statements
onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  if (globeInstance) {
    // Clean up globe instance if necessary
    globeInstance._destructor?.()
  }
})

onMounted(async () => {
  // Wait for DOM to be ready
  await nextTick()

  if (!globeContainer.value) {
    console.error('Globe container not found')
    return
  }

  try {
    // Dynamic import for Globe.gl (better for Vue/SSR)
    const Globe = (await import('globe.gl')).default

    // Fetch the GeoJSON data
    const response = await fetch('/data/countries.geojson')
    if (!response.ok) {
      throw new Error(
        `Failed to fetch data: ${response.status} ${response.statusText}`
      )
    }

    const countries: GeoJSON = await response.json()

    // Initialize the globe with the container
    globeInstance = new Globe(globeContainer.value)
      .width(window.innerWidth)
      .height(window.innerHeight)
      .globeImageUrl('//unpkg.com/three-globe/example/img/earth-dark.jpg')
      .polygonsData(countries.features)
      .polygonCapColor(() => '#888888') // Gray fill color for all countries
      .polygonSideColor(() => '#666666') // Slightly darker gray for the sides
      .polygonStrokeColor(() => '#FFFFFF') // White borders between countries
      .polygonAltitude(0.01) // Small height to create a subtle 3D effect
      .polygonLabel((obj: any) => {
        const d = obj.properties
        return `<b>${d.ADMIN} (${d.ISO_A2})</b>`
      })

    // Make the globe responsive
    window.addEventListener('resize', handleResize)

    console.log('Globe initialized successfully')
  } catch (error) {
    console.error('Error initializing globe:', error)
  }
})
</script>

<template>
  <div ref="globeContainer" class="globe-container"></div>
</template>

<style scoped>
.globe-container {
  width: 100%;
  height: 100vh;
  position: relative;
  overflow: hidden;
  background-color: #000;
  margin: 0;
  padding: 0;
  display: block;
}
</style>
