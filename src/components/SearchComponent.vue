<template>
  <div class="country-search">
    <div
      :class="[
        'dropdown',
        { 'is-active': showDropdown && filteredCountries.length > 0 }
      ]"
    >
      <div
        class="dropdown-menu dropdown-menu-top"
        id="dropdown-menu"
        role="menu"
      >
        <div class="dropdown-content">
          <p
            v-for="(country, index) in filteredCountries"
            :key="index"
            class="dropdown-item"
            :class="{ 'is-active': index === highlightedIndex }"
            v-html="displayFormatted(country.properties.NAME)"
            @click.prevent="selectCountry(country)"
            @mouseover="highlightedIndex = index"
          ></p>
          <p
            v-if="filteredCountries.length === 0 && searchTerm"
            class="dropdown-item"
          >
            No countries found
          </p>
        </div>
      </div>
    </div>

    <div class="field has-addons mb-0">
      <p class="control has-icons-left is-expanded">
        <input
          class="input"
          type="text"
          autocomplete="off"
          v-model="searchTerm"
          placeholder="Search for a country..."
          @focus="showDropdown = true"
          @keydown.down.prevent="highlightNext"
          @keydown.up.prevent="highlightPrev"
          @keydown.enter.prevent="selectHighlighted"
          ref="searchInput"
        />
        <span class="icon is-small is-left">
          <i class="fas fa-search"></i>
        </span>
      </p>
      <div class="control">
        <button
          class="button is-success is-wider"
          @click="confirmSelection"
          :disabled="!searchTerm"
        >
          <span class="icon is-small">
            <i class="fas fa-check"></i>
          </span>
        </button>
      </div>
    </div>

    <div v-if="confirmedCountry" class="notification is-light is-info mt-3">
      <div class="level">
        <div class="level-left">
          <div class="level-item">
            <span class="icon mr-2">
              <i class="fas fa-flag"></i>
            </span>
            <div>
              <p class="heading">Selected Country</p>
              <p class="title is-5">{{ confirmedCountry.properties.NAME }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div
      v-if="showConfirmation"
      class="notification is-light is-success mt-2 py-2"
    >
      <div class="level is-mobile">
        <div class="level-left">
          <div class="level-item">
            <span class="icon">
              <i class="fas fa-check-circle"></i>
            </span>
            <span>Country selection confirmed!</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const searchTerm = ref('')
const countries = ref([])
const selectedCountry = ref(null)
const confirmedCountry = ref(null)
const showDropdown = ref(false)
const loading = ref(false)
const highlightedIndex = ref(-1)
const showConfirmation = ref(false)
const searchInput = ref(null)

onMounted(async () => {
  try {
    loading.value = true
    const response = await fetch('/data/countries.geojson')

    if (!response.ok) {
      throw new Error(`Failed to load countries data: ${response.status}`)
    }

    const data = await response.json()

    // Assuming the GeoJSON has a features array containing country data
    countries.value = data.features || []
  } catch (error) {
    console.error('Error loading countries data:', error)
  } finally {
    loading.value = false
  }
})

const filteredCountries = computed(() => {
  if (!searchTerm.value) {
    highlightedIndex.value = -1
    return []
  }

  const term = searchTerm.value.toLowerCase()

  const results = countries.value
    .filter((country) => country.properties.NAME.toLowerCase().includes(term))
    .sort((a, b) => {
      const aStartsWith = a.properties.NAME.toLowerCase().startsWith(term)
      const bStartsWith = b.properties.NAME.toLowerCase().startsWith(term)

      if (aStartsWith && !bStartsWith) {
        return -1
      }
      if (!aStartsWith && bStartsWith) {
        return 1
      }
      return a.properties.NAME.localeCompare(b.properties.NAME)
    })
    .slice(0, 10) // Limit to 10 results for better performance

  highlightedIndex.value = results.length > 0 ? 0 : -1
  return results
})

function displayFormatted(name) {
  const searchTermLower = searchTerm.value.toLowerCase()
  const index = name.toLowerCase().indexOf(searchTermLower)

  if (index === -1) {
    return name
  }

  const beforeMatch = name.slice(0, index)
  const match = name.slice(index, index + searchTerm.value.length)
  const afterMatch = name.slice(index + searchTerm.value.length)

  return `${beforeMatch}<strong>${match}</strong>${afterMatch}`
}

const selectCountry = (country) => {
  selectedCountry.value = country
  searchTerm.value = country.properties.NAME
  showDropdown.value = false
}

// Down arrow key
function highlightNext() {
  if (highlightedIndex.value < filteredCountries.value.length - 1) {
    highlightedIndex.value++
  }
}

// Up arrow key
function highlightPrev() {
  if (highlightedIndex.value > 0) {
    highlightedIndex.value--
  }
}

// Enter key
function selectHighlighted() {
  if (
    highlightedIndex.value >= 0 &&
    highlightedIndex.value < filteredCountries.value.length
  ) {
    selectCountry(filteredCountries.value[highlightedIndex.value])
  }
}

const confirmSelection = () => {
  if (
    searchTerm.value &&
    (selectedCountry.value || filteredCountries.value.length > 0)
  ) {
    // If there's a selected country, confirm it
    // Otherwise, take the first match if available
    confirmedCountry.value = selectedCountry.value || filteredCountries.value[0]
    showConfirmation.value = true

    // Hide confirmation message after 3 seconds
    setTimeout(() => {
      showConfirmation.value = false
    }, 3000)
  }
}

// Close dropdown when clicking outside
const handleClickOutside = (event) => {
  if (searchInput.value && !searchInput.value.contains(event.target)) {
    showDropdown.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

// Reset the selected country when search term changes
watch(searchTerm, () => {
  if (
    !selectedCountry.value ||
    searchTerm.value !== selectedCountry.value.properties.NAME
  ) {
    selectedCountry.value = null
  }

  if (searchTerm.value) {
    showDropdown.value = true
  }
})
</script>

<style scoped>
.dropdown {
  width: 100%;
}

.dropdown-menu {
  width: 100%;
  padding-top: 0;
}

.dropdown-item.is-active {
  background-color: #f5f5f5;
  color: #000;
}

.notification {
  padding: 1rem;
}

/* Position dropdown above the input */
.dropdown-menu.dropdown-menu-top {
  bottom: 100%;
  top: auto;
  margin-top: 0;
  margin-bottom: 5px;
  box-shadow:
    0 -2px 3px rgba(10, 10, 10, 0.1),
    0 0 0 1px rgba(10, 10, 10, 0.1);
}

/* Ensure dropdown doesn't disappear on hover */
.dropdown.is-active .dropdown-menu {
  display: block;
}

/* Make the checkmark button wider */
.button.is-wider {
  min-width: 60px;
}
</style>
