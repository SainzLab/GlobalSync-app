<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const currentYear = ref(new Date().getFullYear()); 
const countryCode = ref('ID'); 
const holidays = ref({});
const isLoading = ref(false);

const tooltip = ref({
  visible: false,
  text: '',
  style: { top: '0px', left: '0px' }
});

const monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
const daysHeader = ["S", "M", "T", "W", "T", "F", "S"];

const fetchHolidays = async () => {
  isLoading.value = true;
  holidays.value = {}; 
  try {
    const response = await fetch(`https://date.nager.at/api/v3/publicholidays/${currentYear.value}/${countryCode.value}`);
    if (!response.ok) throw new Error('Failed to fetch');
    const data = await response.json();
    
    data.forEach(h => {
      holidays.value[h.date] = h.localName; 
    });
  } catch (error) {
    console.error("Error fetching holidays:", error);
  } finally {
    setTimeout(() => { isLoading.value = false; }, 300);
  }
};

onMounted(fetchHolidays);
watch(currentYear, fetchHolidays);

const changeYear = (delta) => { currentYear.value += delta; };

const yearCalendar = computed(() => {
  return monthNames.map((name, index) => {
    const firstDay = new Date(currentYear.value, index, 1).getDay();
    const daysInMonth = new Date(currentYear.value, index + 1, 0).getDate();
    return { 
      name, 
      index, 
      padding: Array(firstDay).fill(null), 
      days: Array.from({ length: daysInMonth }, (_, i) => i + 1) 
    };
  });
});

const isToday = (day, monthIndex) => {
  const today = new Date();
  return today.getDate() === day && today.getMonth() === monthIndex && today.getFullYear() === currentYear.value; 
};

const getHolidayName = (day, monthIndex) => {
  const monthStr = String(monthIndex + 1).padStart(2, '0');
  const dayStr = String(day).padStart(2, '0');
  const dateKey = `${currentYear.value}-${monthStr}-${dayStr}`;
  return holidays.value[dateKey];
};

const showHolidayTooltip = (event, holidayName) => {
  if (!holidayName) return;
  
  const rect = event.target.getBoundingClientRect();
  
  tooltip.value = {
    visible: true,
    text: holidayName,
    style: {
      top: `${rect.top - 45}px`, 
      left: `${rect.left + (rect.width / 2)}px`
    }
  };
};

const hideTooltip = () => {
  tooltip.value.visible = false;
};
</script>

<template>
  <div class="calendar-container">
    <div class="year-nav">
      <button @click="changeYear(-1)" class="nav-btn">❮</button>
      
      <div class="title-wrapper">
        <h2 :key="currentYear" class="year-title">{{ currentYear }}</h2>
        <span class="sub-title">INDONESIA CALENDAR</span>
      </div>

      <button @click="changeYear(1)" class="nav-btn">❯</button>
    </div>
    
    <div v-if="isLoading" class="loading-state">
      <div class="spinner"></div>
      <p>Fetching Holidays...</p>
    </div>

    <div v-else class="months-grid">
      <div 
        v-for="(month, mIndex) in yearCalendar" 
        :key="month.name + currentYear" 
        class="month-card"
        :style="{ animationDelay: `${mIndex * 50}ms` }"
      >
        <h3>{{ month.name }}</h3>
        
        <div class="days-head">
          <span v-for="d in daysHeader" :key="d" :class="{'is-weekend': d === 'S'}">{{ d }}</span>
        </div>
        
        <div class="dates-grid">
          <span v-for="(p, i) in month.padding" :key="'p'+i"></span>
          
          <span 
            v-for="d in month.days" 
            :key="'d'+d" 
            class="date-item" 
            :class="{ 
              'today': isToday(d, mIndex),
              'is-holiday': !!getHolidayName(d, mIndex)
            }"
            @mouseenter="(e) => showHolidayTooltip(e, getHolidayName(d, mIndex))"
            @mouseleave="hideTooltip"
          >
            {{ d }}
            <span v-if="getHolidayName(d, mIndex)" class="holiday-dot"></span>
          </span>
        </div>
      </div>
    </div>

    <Transition name="fade">
      <div 
        v-if="tooltip.visible" 
        class="holiday-popup" 
        :style="tooltip.style"
      >
        {{ tooltip.text }}
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.calendar-container { 
  padding: 2rem; 
  width: 100%; 
  box-sizing: border-box; 
  max-width: 1400px; 
  margin: 0 auto; 
  color: #eee; 
  font-family: 'Segoe UI', sans-serif; 
}

.year-nav { display: flex; justify-content: space-between; align-items: center; margin-bottom: 3rem; width: 100%; position: relative; }
.title-wrapper { display: flex; flex-direction: column; align-items: center; }
.year-nav h2.year-title { font-size: 3.5rem; color: #fff; margin: 0; font-weight: 900; letter-spacing: 5px; text-shadow: 0 4px 20px rgba(234, 88, 12, 0.4); animation: popIn 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.sub-title { font-size: 0.8rem; color: #888; letter-spacing: 4px; margin-top: -5px; }

.nav-btn { 
  background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); 
  color: white; width: 50px; height: 50px; border-radius: 50%; cursor: pointer; 
  font-size: 1.2rem; transition: all 0.2s ease; display: flex; align-items: center; justify-content: center;
}
.nav-btn:hover { background: #ea580c; border-color: #ea580c; transform: scale(1.1); box-shadow: 0 0 15px rgba(234, 88, 12, 0.5); }

.months-grid { display: grid; grid-template-columns: 1fr; gap: 2rem; }
.month-card { 
  background: rgba(30, 30, 30, 0.6); 
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255,255,255,0.08); 
  border-radius: 16px; padding: 1.2rem; 
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  opacity: 0; 
  animation: slideUpFade 0.6s cubic-bezier(0.2, 0.8, 0.2, 1) forwards;
}

.month-card:hover { transform: translateY(-5px); box-shadow: 0 10px 30px rgba(0,0,0,0.5); border-color: rgba(234, 88, 12, 0.3); }
.month-card h3 { color: #ea580c; text-align: center; margin: 0 0 15px 0; letter-spacing: 2px; font-size: 1.1rem; text-transform: uppercase; font-weight: 700; }

.days-head { display: grid; grid-template-columns: repeat(7, 1fr); margin-bottom: 10px; border-bottom: 1px solid rgba(255,255,255,0.05); padding-bottom: 5px; }
.days-head span { text-align: center; font-size: 0.75rem; color: #666; font-weight: bold; }
.days-head span.is-weekend { color: #b91c1c; }

.dates-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px; }
.date-item { 
  position: relative;
  text-align: center; font-size: 0.9rem; color: #ccc; 
  padding: 8px 0; border-radius: 8px; cursor: default;
  transition: background 0.2s;
}

.date-item:hover { background: rgba(255,255,255,0.1); color: #fff; }

.today { background: linear-gradient(135deg, #ea580c, #c2410c) !important; color: white !important; font-weight: bold; box-shadow: 0 4px 10px rgba(234, 88, 12, 0.4); }

.is-holiday { color: #f87171; font-weight: 600; cursor: help; }
.is-holiday:hover { background: rgba(220, 38, 38, 0.15); }

.holiday-dot {
  position: absolute; bottom: 4px; left: 50%; transform: translateX(-50%);
  width: 4px; height: 4px; background-color: #f87171; border-radius: 50%;
}
.today .holiday-dot { background-color: white; }

.holiday-popup {
  position: fixed;
  background: rgba(20, 20, 20, 0.95);
  color: #fff;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 500;
  pointer-events: none; 
  transform: translate(-50%, 0); 
  z-index: 9999;
  border: 1px solid #ea580c;
  box-shadow: 0 4px 15px rgba(0,0,0,0.5);
  white-space: nowrap;
}

.holiday-popup::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  margin-left: -5px;
  border-width: 5px;
  border-style: solid;
  border-color: #ea580c transparent transparent transparent;
}

.fade-enter-active,
.fade-leave-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.fade-enter-from,
.fade-leave-to { opacity: 0; transform: translate(-50%, 10px); }

.loading-state { grid-column: 1 / -1; display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 4rem; opacity: 0.7; }
.spinner { width: 40px; height: 40px; border: 4px solid rgba(255,255,255,0.1); border-left-color: #ea580c; border-radius: 50%; animation: spin 1s linear infinite; margin-bottom: 1rem; }

@keyframes slideUpFade {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes popIn {
  0% { transform: scale(0.8); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}
@keyframes spin { 100% { transform: rotate(360deg); } }

@media (min-width: 768px) {
  .months-grid { grid-template-columns: repeat(2, 1fr); }
}
@media (min-width: 1200px) {
  .months-grid { grid-template-columns: repeat(4, 1fr); }
}
</style>