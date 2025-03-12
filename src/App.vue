<template>
  <div class="app-container">
    <!-- Header placeholder component -->
    <header class="app-header">
      <h1>GeoHearo</h1>
    </header>

    <!-- Main content area with globe -->
    <main class="main-content">
      <GlobeViz class="globe-section" />
    </main>

    <SearchComponent />

    <!-- Footer placeholder component -->
    <!-- <footer class="app-footer"></footer> -->
  </div>
</template>

<script setup lang="ts">
import GlobeViz from './components/GlobeViz.vue'
import SearchComponent from './components/SearchComponent.vue'
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100%;
  overflow: hidden;
}

.app-header {
  background-color: #1a1a2e;
  color: white;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 70px;
}

.header-nav {
  display: flex;
  gap: 1rem;
}

.nav-button {
  background-color: #0f3460;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.nav-button:hover {
  background-color: #16213e;
}

.main-content {
  flex: 1;
  position: relative;
  overflow: hidden;
}

.globe-section {
  width: 100%;
  height: 100%;
}

.app-footer {
  background-color: #1a1a2e;
  color: white;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 60px;
}

.footer-info {
  font-size: 0.9rem;
}

.footer-controls {
  display: flex;
  gap: 1rem;
}

.footer-button {
  background-color: #0f3460;
  color: white;
  border: none;
  padding: 0.4rem 0.8rem;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
  font-size: 0.9rem;
}

.footer-button:hover {
  background-color: #16213e;
}
</style>
