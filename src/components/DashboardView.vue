<script setup>
import { ref, computed } from 'vue';
import NeoClock from './NeoClock.vue';

const props = defineProps({
  searchQuery: String,
  currentTime: Date
});

const cities = ref([
  { id: 1, city: 'Jakarta', zone: 'Asia/Jakarta', offset: '+0H' },
  { id: 2, city: 'Makassar', zone: 'Asia/Makassar', offset: '+1H' },
  { id: 3, city: 'Jayapura', zone: 'Asia/Jayapura', offset: '+2H' },
  { id: 4, city: 'London', zone: 'Europe/London', offset: '-7H' },
  { id: 5, city: 'New York', zone: 'America/New_York', offset: '-12H' },
  { id: 6, city: 'San Francisco', zone: 'America/Los_Angeles', offset: '-15H' },
  { id: 7, city: 'Tokyo', zone: 'Asia/Tokyo', offset: '+2H' },
  { id: 8, city: 'Dubai', zone: 'Asia/Dubai', offset: '-3H' },
]);

const filteredCities = computed(() => {
  if (!props.searchQuery) return cities.value;
  return cities.value.filter(item => 
    item.city.toLowerCase().includes(props.searchQuery.toLowerCase())
  );
});
</script>

<template>
  <div class="dashboard-grid">
    <div 
      v-for="city in filteredCities" 
      :key="city.id"
      class="grid-item"
    >
      <NeoClock v-bind="city" :time="currentTime" />
    </div>

    <div v-if="filteredCities.length === 0" class="no-result">
      <p>SIGNAL LOST</p>
      <small>City "{{ searchQuery }}" not found.</small>
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
  text-align: center;
  padding: 3rem;
  color: #ea580c;
  border: 1px dashed #333;
  border-radius: 12px;
}

@media (min-width: 1024px) {
  .dashboard-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 2.5rem; 
    padding-top: 3rem;
    padding-bottom: 4rem;
  }
}

@media (min-width: 1600px) {
  .dashboard-grid {
    transform: scale(1.1); 
    margin-top: 2rem;
  }
}
</style>