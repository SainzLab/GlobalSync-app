<script setup>
import { ref, computed } from 'vue';

const currentYear = ref(new Date().getFullYear()); 
const monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
const daysHeader = ["S", "M", "T", "W", "T", "F", "S"];

const changeYear = (delta) => { currentYear.value += delta; };

const yearCalendar = computed(() => {
  return monthNames.map((name, index) => {
    const firstDay = new Date(currentYear.value, index, 1).getDay();
    const daysInMonth = new Date(currentYear.value, index + 1, 0).getDate();
    return { 
      name, 
      padding: Array(firstDay).fill(null), 
      days: Array.from({ length: daysInMonth }, (_, i) => i + 1) 
    };
  });
});

const isToday = (day, monthIndex) => {
  const today = new Date();
  return today.getDate() === day && today.getMonth() === monthIndex && today.getFullYear() === currentYear.value; 
};
</script>

<template>
  <div class="calendar-container">
    <div class="year-nav">
      <button @click="changeYear(-1)">❮</button>
      <h2>{{ currentYear }} OVERVIEW</h2>
      <button @click="changeYear(1)">❯</button>
    </div>
    
    <div class="months-grid">
      <div v-for="(month, mIndex) in yearCalendar" :key="mIndex" class="month-card">
        <h3>{{ month.name }}</h3>
        <div class="days-head">
          <span v-for="d in daysHeader" :key="d">{{ d }}</span>
        </div>
        <div class="dates-grid">
          <span v-for="(p, i) in month.padding" :key="'p'+i"></span>
          <span v-for="d in month.days" :key="'d'+d" class="date-item" :class="{ 'today': isToday(d, mIndex) }">
            {{ d }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar-container { padding: 1rem 1rem 4rem 1rem; width: 100%; box-sizing: border-box; max-width: 1400px; margin: 0 auto; }
.year-nav { display: flex; justify-content: space-between; align-items: center; margin-bottom: 2rem; width: 100%; }
.year-nav h2 { font-size: 1.5rem; color: #333; margin: 0; font-weight: 800; letter-spacing: 2px; text-align: center; flex: 1; }
.year-nav button { background: rgba(255,255,255,0.1); border: none; color: white; width: 44px; height: 44px; border-radius: 50%; cursor: pointer; font-size: 1.2rem; }

.months-grid { display: grid; grid-template-columns: 1fr; gap: 1.5rem; }
.month-card { background: rgba(20,20,20,0.6); border: 1px solid #333; border-radius: 12px; padding: 1rem; }
.month-card h3 { color: #ea580c; text-align: center; margin: 0 0 10px 0; letter-spacing: 1px; font-size: 1rem;}

.days-head { display: grid; grid-template-columns: repeat(7, 1fr); margin-bottom: 5px; }
.days-head span { text-align: center; font-size: 0.7rem; color: #666; font-weight: bold; }

.dates-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 2px; }
.date-item { text-align: center; font-size: 0.8rem; color: #aaa; padding: 4px 0; }
.today { background: #ea580c; color: white; border-radius: 4px; font-weight: bold; }

@media (min-width: 1024px) {
  .months-grid { grid-template-columns: repeat(4, 1fr); }
  .year-nav h2 { font-size: 3rem; letter-spacing: 10px; }
}
</style>