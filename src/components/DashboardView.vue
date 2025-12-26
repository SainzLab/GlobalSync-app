<script setup>
import { computed } from 'vue';
import NeoClock from './NeoClock.vue';

const props = defineProps({
  searchQuery: String,
  citiesData: {
    type: Array,
    required: true,
    default: () => []
  }
});

const filteredCities = computed(() => {
  if (!props.searchQuery) return props.citiesData;
  
  const query = props.searchQuery.toLowerCase();
  
  return props.citiesData.filter(item => {

    const matchCity = item.city.toLowerCase().includes(query);
    const matchCountry = item.country.toLowerCase().includes(query);
    const matchKeywords = item.keywords.some(k => k.toLowerCase().includes(query));
    
    return matchCity || matchCountry || matchKeywords;
  });
});
</script>

<template>
  <div class="dashboard-grid">
    <div 
      v-for="city in filteredCities" 
      :key="city.id"
      class="grid-item"
    >
      <NeoClock 
        :time="city.time"
        :timezone="city.zone"
        :city="city.city"
        :offset="city.offset"
      />
    </div>

    <div v-if="filteredCities.length === 0" class="no-result">
      <div class="alert-icon">!</div>
      <p>NO SIGNAL DETECTED</p>
      <small>Target "{{ searchQuery }}" is out of range.</small>
    </div>
  </div>
</template>

<style scoped>
.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr); 
  gap: 12px;
  width: 100%;
  padding: 0 16px 64px 16px;
  box-sizing: border-box;
  max-width: 1100px; 
  margin: 0 auto;
}

.no-result {
  grid-column: 1 / -1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem 2rem;
  color: #ea580c;
  border: 1px dashed rgba(234, 88, 12, 0.3);
  border-radius: 16px;
  background: rgba(234, 88, 12, 0.05);
  margin-top: 2rem;
}

.alert-icon {
  font-size: 2rem; font-weight: bold; margin-bottom: 1rem;
  width: 50px; height: 50px; border-radius: 50%; border: 2px solid #ea580c;
  display: flex; align-items: center; justify-content: center;
}

.no-result p {
  font-weight: 800; letter-spacing: 2px; margin: 0; font-size: 1.2rem;
}
.no-result small {
  color: #666; margin-top: 0.5rem; font-family: monospace;
}

@media (min-width: 1024px) {
  .dashboard-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 2.5rem; 
    padding-top: 3rem;
    padding-bottom: 4rem;
  }
}
</style>