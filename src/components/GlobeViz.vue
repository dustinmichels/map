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
const hoveredCountry = ref<string | null>(null)
const isLoading = ref(true) // Add loading state

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
    isLoading.value = false // Stop loading if container not found
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
      .polygonCapColor((d: any) => {
        // Change color when hovered
        return d.properties?.ISO_A2 === hoveredCountry.value
          ? '#FF8C00'
          : '#888888'
      })
      .polygonSideColor((d: any) => {
        // Change side color when hovered
        return d.properties?.ISO_A2 === hoveredCountry.value
          ? '#E47200'
          : '#666666'
      })
      .polygonStrokeColor(() => '#FFFFFF') // White borders between countries
      .polygonAltitude((d: any) => {
        // Slightly elevate the hovered country
        return d.properties?.ISO_A2 === hoveredCountry.value ? 0.02 : 0.01
      })
      .polygonLabel((obj: any) => {
        const d = obj.properties
        return `<b>${d.ADMIN}</b>`
      })
      .onPolygonHover((polygon: any) => {
        // Update the hovered country and re-render the globe
        if (polygon) {
          hoveredCountry.value = polygon.properties.ISO_A2
        } else {
          hoveredCountry.value = null
        }

        // Force update polygon colors
        globeInstance
          .polygonCapColor((d: any) =>
            d.properties?.ISO_A2 === hoveredCountry.value
              ? '#FF8C00'
              : '#888888'
          )
          .polygonSideColor((d: any) =>
            d.properties?.ISO_A2 === hoveredCountry.value
              ? '#E47200'
              : '#666666'
          )
          .polygonAltitude((d: any) =>
            d.properties?.ISO_A2 === hoveredCountry.value ? 0.02 : 0.01
          )
      })

    // Make the globe responsive
    window.addEventListener('resize', handleResize)

    // Add a small delay to ensure the globe is fully rendered
    setTimeout(() => {
      isLoading.value = false // Hide loading animation
    }, 1000)

    console.log('Globe initialized successfully')
  } catch (error) {
    console.error('Error initializing globe:', error)
    isLoading.value = false // Stop loading on error
  }
})
</script>

<template>
  <div class="globe-wrapper">
    <!-- Loading overlay with spinner -->
    <div v-if="isLoading" class="loading-overlay">
      <div class="spinner"></div>
      <div class="loading-text">Loading Globe...</div>
    </div>

    <!-- Globe container -->
    <div ref="globeContainer" class="globe-container"></div>
  </div>
</template>

<style scoped>
.globe-wrapper {
  width: 100%;
  height: 100vh;
  position: relative;
  overflow: hidden;
}

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

/* Loading overlay styling */
.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.loading-text {
  color: white;
  margin-top: 16px;
  font-size: 18px;
  font-family: Arial, sans-serif;
}

/* CSS Spinner */
.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: #ff8c00;
  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
