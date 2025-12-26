<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';
import DashboardView from './components/DashboardView.vue';
import CalendarView from './components/CalendarView.vue';

// --- STATE MANAGEMENT ---
const isCalendarOpen = ref(false); 
const searchQuery = ref('');
const currentPage = ref(1);
const windowWidth = ref(window.innerWidth);

// Boot Sequence State
const isBooting = ref(true);
const bootStep = ref(0);
const bootMessages = [
  'INITIALIZING KERNEL...',
  'ESTABLISHING UPLINK...',
  'SYNCING ATOMIC CLOCKS...',
  'DECRYPTING GEO-DATA...',
  'SYSTEM READY.'
];

// Search Scanning State
const isSearching = ref(false);
let searchTimeout = null;
let ticker = null;

// --- DATA ---
const cities = ref([
  { id: 1, city: 'Jakarta', country: 'Indonesia', keywords: ['java', 'jawa', 'wib'], zone: 'Asia/Jakarta', time: null, offset: '+7' },
  { id: 2, city: 'Makassar', country: 'Indonesia', keywords: ['sulawesi', 'wita'], zone: 'Asia/Makassar', time: null, offset: '+8' },
  { id: 3, city: 'Jayapura', country: 'Indonesia', keywords: ['papua', 'wit'], zone: 'Asia/Jayapura', time: null, offset: '+9' },
  { id: 4, city: 'Tokyo', country: 'Japan', keywords: ['jepang', 'osaka'], zone: 'Asia/Tokyo', time: null, offset: '+9' },
  { id: 5, city: 'Singapore', country: 'Singapore', keywords: ['sg', 'malay'], zone: 'Asia/Singapore', time: null, offset: '+8' },
  { id: 6, city: 'Bangkok', country: 'Thailand', keywords: ['thai', 'asia'], zone: 'Asia/Bangkok', time: null, offset: '+7' },
  { id: 7, city: 'Seoul', country: 'South Korea', keywords: ['korea', 'kpop'], zone: 'Asia/Seoul', time: null, offset: '+9' },
  { id: 8, city: 'Shanghai', country: 'China', keywords: ['cn', 'beijing'], zone: 'Asia/Shanghai', time: null, offset: '+8' },
  { id: 9, city: 'Mumbai', country: 'India', keywords: ['delhi', 'ind'], zone: 'Asia/Kolkata', time: null, offset: '+5.5' },
  { id: 10, city: 'Dubai', country: 'UAE', keywords: ['arab', 'emirates'], zone: 'Asia/Dubai', time: null, offset: '+4' },
  { id: 11, city: 'Sydney', country: 'Australia', keywords: ['aussie', 'oceania'], zone: 'Australia/Sydney', time: null, offset: '+11' },
  { id: 12, city: 'Auckland', country: 'New Zealand', keywords: ['nz', 'kiwi'], zone: 'Pacific/Auckland', time: null, offset: '+13' },
  { id: 13, city: 'Riyadh', country: 'Saudi Arabia', keywords: ['saudi', 'mecca'], zone: 'Asia/Riyadh', time: null, offset: '+3' },
  { id: 14, city: 'Hong Kong', country: 'Hong Kong', keywords: ['hk', 'asia'], zone: 'Asia/Hong_Kong', time: null, offset: '+8' },
  { id: 15, city: 'Manila', country: 'Philippines', keywords: ['pinoy'], zone: 'Asia/Manila', time: null, offset: '+8' },
  { id: 16, city: 'London', country: 'United Kingdom', keywords: ['uk', 'england'], zone: 'Europe/London', time: null, offset: '+0' },
  { id: 17, city: 'Paris', country: 'France', keywords: ['fr', 'europe'], zone: 'Europe/Paris', time: null, offset: '+1' },
  { id: 18, city: 'Berlin', country: 'Germany', keywords: ['de', 'ger'], zone: 'Europe/Berlin', time: null, offset: '+1' },
  { id: 19, city: 'Moscow', country: 'Russia', keywords: ['ru', 'soviet'], zone: 'Europe/Moscow', time: null, offset: '+3' },
  { id: 20, city: 'Rome', country: 'Italy', keywords: ['it', 'vatican'], zone: 'Europe/Rome', time: null, offset: '+1' },
  { id: 21, city: 'Madrid', country: 'Spain', keywords: ['es', 'espana'], zone: 'Europe/Madrid', time: null, offset: '+1' },
  { id: 22, city: 'Amsterdam', country: 'Netherlands', keywords: ['nl', 'dutch'], zone: 'Europe/Amsterdam', time: null, offset: '+1' },
  { id: 23, city: 'Kyiv', country: 'Ukraine', keywords: ['ua', 'eastern'], zone: 'Europe/Kyiv', time: null, offset: '+2' },
  { id: 24, city: 'Istanbul', country: 'Turkey', keywords: ['tr', 'turkiye'], zone: 'Europe/Istanbul', time: null, offset: '+3' },
  { id: 25, city: 'Zurich', country: 'Switzerland', keywords: ['ch', 'swiss'], zone: 'Europe/Zurich', time: null, offset: '+1' },
  { id: 26, city: 'New York', country: 'USA', keywords: ['ny', 'est'], zone: 'America/New_York', time: null, offset: '-5' },
  { id: 27, city: 'Los Angeles', country: 'USA', keywords: ['ca', 'pst'], zone: 'America/Los_Angeles', time: null, offset: '-8' },
  { id: 28, city: 'Chicago', country: 'USA', keywords: ['il', 'cst'], zone: 'America/Chicago', time: null, offset: '-6' },
  { id: 29, city: 'Toronto', country: 'Canada', keywords: ['ca', 'ontario'], zone: 'America/Toronto', time: null, offset: '-5' },
  { id: 30, city: 'Mexico City', country: 'Mexico', keywords: ['mx', 'latam'], zone: 'America/Mexico_City', time: null, offset: '-6' },
  { id: 31, city: 'Sao Paulo', country: 'Brazil', keywords: ['br', 'latam'], zone: 'America/Sao_Paulo', time: null, offset: '-3' },
  { id: 32, city: 'Buenos Aires', country: 'Argentina', keywords: ['ar', 'latam'], zone: 'America/Argentina/Buenos_Aires', time: null, offset: '-3' },
  { id: 33, city: 'Vancouver', country: 'Canada', keywords: ['bc', 'west'], zone: 'America/Vancouver', time: null, offset: '-8' },
  { id: 34, city: 'Lima', country: 'Peru', keywords: ['pe', 'latam'], zone: 'America/Lima', time: null, offset: '-5' },
  { id: 35, city: 'Bogota', country: 'Colombia', keywords: ['co', 'latam'], zone: 'America/Bogota', time: null, offset: '-5' },
  { id: 36, city: 'Cairo', country: 'Egypt', keywords: ['eg', 'africa'], zone: 'Africa/Cairo', time: null, offset: '+2' },
  { id: 38, city: 'Lagos', country: 'Nigeria', keywords: ['ng', 'africa'], zone: 'Africa/Lagos', time: null, offset: '+1' },
  { id: 39, city: 'Nairobi', country: 'Kenya', keywords: ['ke', 'africa'], zone: 'Africa/Nairobi', time: null, offset: '+3' },
  { id: 40, city: 'Casablanca', country: 'Morocco', keywords: ['ma', 'maghreb'], zone: 'Africa/Casablanca', time: null, offset: '+1' },
]);

// --- LOGIC ---

// Responsive
const updateWidth = () => { windowWidth.value = window.innerWidth; };
const itemsPerPage = computed(() => windowWidth.value >= 1024 ? 16 : 8);

// Search with Fake Delay
const handleSearchInput = (e) => {
  isSearching.value = true;
  currentPage.value = 1;
  
  if (searchTimeout) clearTimeout(searchTimeout);

  searchTimeout = setTimeout(() => {
    isSearching.value = false;
  }, 600); // 600ms fake delay
};

// Filter & Pagination
const filteredCities = computed(() => {
  const query = searchQuery.value.toLowerCase().trim();
  if (!query) return cities.value;
  
  return cities.value.filter(c => 
    c.city.toLowerCase().includes(query) || 
    c.country.toLowerCase().includes(query) ||
    (c.keywords && c.keywords.some(k => k.toLowerCase().includes(query)))
  );
});

const totalPages = computed(() => Math.ceil(filteredCities.value.length / itemsPerPage.value));
const paginatedCities = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return filteredCities.value.slice(start, end);
});

const prevPage = () => { if (currentPage.value > 1) currentPage.value--; };
const nextPage = () => { if (currentPage.value < totalPages.value) currentPage.value++; };

// Clock Logic
const updateTimes = () => {
  const now = new Date();
  cities.value.forEach(city => {
    const timeString = now.toLocaleString('en-US', { timeZone: city.zone });
    city.time = new Date(timeString);
  });
};

// Boot Animation Logic
const runBootSequence = () => {
  let step = 0;
  const interval = setInterval(() => {
    bootStep.value = step;
    step++;
    if (step >= bootMessages.length) {
      clearInterval(interval);
      setTimeout(() => {
        isBooting.value = false;
      }, 800);
    }
  }, 400); 
};

onMounted(() => {
  window.addEventListener('resize', updateWidth);
  updateTimes();
  runBootSequence();
  ticker = setInterval(() => { updateTimes(); }, 1000);
});

onUnmounted(() => {
  window.removeEventListener('resize', updateWidth);
  clearInterval(ticker);
  if (searchTimeout) clearTimeout(searchTimeout);
});
</script>

<template>
  <div class="app-container">
    <div class="scanlines"></div> 
    
    <transition name="crt-fade">
      <div v-if="isBooting" class="boot-overlay">
        <div class="terminal-boot">
          <div v-for="(msg, index) in bootMessages" :key="index" class="boot-line">
            <span v-if="index <= bootStep" :class="{'blinking': index === bootStep}">
              > {{ msg }}
            </span>
          </div>
          <div class="boot-progress">
             <div class="progress-bar" :style="{ width: (bootStep + 1) * 20 + '%' }"></div>
          </div>
        </div>
      </div>
    </transition>

    <header class="navbar">
      <div class="brand">
        <h1>GLOBAL<span class="accent">SYNC</span></h1>
        <div class="status-dot"></div>
      </div>
      
      <div class="nav-controls">
        <transition name="slide-fade">
          <div v-if="!isCalendarOpen" class="search-container">
            <span class="input-prefix">CMD:/></span>
            <input 
              type="text" 
              v-model="searchQuery" 
              @input="handleSearchInput"
              placeholder="TARGET_SYSTEM..." 
              class="terminal-input"
            />
            <div v-if="isSearching" class="search-indicator">SCANNING...</div>
          </div>
        </transition>

        <button class="cyber-toggle" :class="{ 'active': isCalendarOpen }" @click="isCalendarOpen = !isCalendarOpen">
          <span class="toggle-label">MODE</span>
          <div class="toggle-switch">
            <span class="state-text">{{ isCalendarOpen ? 'CALENDAR' : 'DASHBOARD' }}</span>
          </div>
        </button>
      </div>
    </header>

    <div class="map-background">
      <img src="/world-map.svg" class="world-map-img" />
      <div class="fade-overlay"></div>
    </div>

    <div class="content-area">
      <transition name="zoom-fade" mode="out-in">
        <div v-if="!isBooting" :key="isCalendarOpen ? 'cal' : 'dash'" class="view-wrapper">
          
          <component 
            :is="isCalendarOpen ? CalendarView : DashboardView"
            :searchQuery="searchQuery"
            :citiesData="paginatedCities"
            :class="{ 'opacity-50': isSearching }" 
            class="component-view"
          />

          <div class="pagination-controls" v-if="!isCalendarOpen && totalPages > 1 && !isSearching">
            <button class="page-btn" @click="prevPage" :disabled="currentPage === 1">&lt;</button>
            <span class="page-info"><span class="accent">{{ currentPage }}</span> / {{ totalPages }}</span>
            <button class="page-btn" @click="nextPage" :disabled="currentPage === totalPages">&gt;</button>
          </div>
          
          <div v-if="filteredCities.length === 0 && !isSearching" class="no-results">
            <p class="blink-text">ERROR: TARGET "{{ searchQuery }}" NOT FOUND</p>
          </div>

        </div>
      </transition>
    </div>

    <footer class="footer">
      <div class="footer-inner">
        <span class="terminal-text">SYS_STATUS: ONLINE</span>
        <span class="terminal-text">©2025 SainzLab</span>
      </div>
    </footer>

  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&family=JetBrains+Mono:wght@400;700&display=swap');
body { margin: 0; background: #050505; color: white; font-family: 'Inter', sans-serif; overflow-x: hidden; }
</style>

<style scoped>

.app-container { min-height: 100vh; display: flex; flex-direction: column; position: relative; }
.scanlines {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 999;
  background: linear-gradient(to bottom, rgba(255,255,255,0), rgba(255,255,255,0) 50%, rgba(0,0,0,0.2) 50%, rgba(0,0,0,0.2));
  background-size: 100% 4px;
}

.navbar {
  position: sticky; top: 0; z-index: 50; 
  background: rgba(10, 10, 10, 0.9);
  border-bottom: 1px solid rgba(234, 88, 12, 0.3);
  box-shadow: 0 5px 20px rgba(234, 88, 12, 0.1);
  padding: 1rem 1.5rem;
  display: flex; flex-direction: column; gap: 1rem;
  backdrop-filter: blur(12px);
}

.brand { display: flex; align-items: center; gap: 10px; justify-content: center; }
h1 { 
  font-size: 1.5rem; margin: 0; font-weight: 800; color: #e5e5e5; letter-spacing: 2px; 
  text-shadow: 0 0 10px rgba(234, 88, 12, 0.5);
}
.accent { color: #ea580c; }
.status-dot { width: 8px; height: 8px; background: #00ff41; border-radius: 50%; box-shadow: 0 0 8px #00ff41; }

.nav-controls { display: flex; width: 100%; align-items: center; min-height: 42px; position: relative; }
.search-container { flex: 1; margin-right: 15px; position: relative; display: flex; align-items: center; }
.input-prefix { 
  position: absolute; left: 10px; font-family: 'JetBrains Mono'; font-size: 0.8rem; color: #666; pointer-events: none; 
}

.terminal-input {
  width: 100%; background: rgba(0, 0, 0, 0.6); 
  border: 1px solid #333; border-left: 2px solid #ea580c;
  color: #ea580c; padding: 10px 12px 10px 70px; 
  font-family: 'JetBrains Mono', monospace; font-size: 0.85rem;
  outline: none; transition: all 0.3s ease;
}
.terminal-input:focus { border-color: #ea580c; box-shadow: 0 0 15px rgba(234, 88, 12, 0.2); }

.search-indicator {
  position: absolute; right: 10px; font-family: 'JetBrains Mono'; font-size: 0.7rem; color: #ea580c;
  animation: blink 0.5s infinite;
}

.cyber-toggle {
  margin-left: auto; 
  display: flex; 
  align-items: center; 
  gap: 12px; 
  background: rgba(255, 255, 255, 0.03); 
  border: 1px solid #333; 
  padding: 4px 6px 4px 12px; 
  border-radius: 4px; 
  cursor: pointer; 
  font-family: 'JetBrains Mono', monospace; 
  transition: all 0.3s ease;
}

.toggle-label { 
  font-size: 0.7rem; 
  color: #a1a1aa; 
  font-weight: 700; 
  letter-spacing: 1px;
  text-transform: uppercase;
}

.cyber-toggle:hover { border-color: #666; background: rgba(255, 255, 255, 0.05); }
.cyber-toggle:hover .toggle-label { color: #e5e5e5; }

.toggle-switch {
  background: #111; padding: 6px 12px; color: #666; font-size: 0.8rem; min-width: 80px; text-align: center;
  border: 1px solid transparent; transition: all 0.3s;
}

.cyber-toggle.active { border-color: #ea580c; }
.cyber-toggle.active .toggle-label { color: #ea580c; }
.cyber-toggle.active .toggle-switch { 
  background: rgba(234, 88, 12, 0.15); color: #ea580c; border-color: #ea580c; 
  box-shadow: 0 0 10px rgba(234, 88, 12, 0.3); text-shadow: 0 0 5px #ea580c;
}

.boot-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  background: #000; z-index: 9999; display: flex; align-items: center; justify-content: center;
}
.terminal-boot { width: 300px; font-family: 'JetBrains Mono', monospace; }
.boot-line { color: #00ff41; font-size: 0.9rem; margin-bottom: 5px; height: 1.2rem; }
.blinking { animation: blink 0.2s infinite; }
.boot-progress { width: 100%; height: 4px; background: #111; margin-top: 20px; }
.progress-bar { height: 100%; background: #ea580c; transition: width 0.4s ease; }

.content-area { flex: 1; position: relative; z-index: 10; padding-top: 1rem; display: flex; flex-direction: column; }
.view-wrapper { width: 100%; display: flex; flex-direction: column; flex: 1; }
.component-view { transition: opacity 0.3s ease; }
.opacity-50 { opacity: 0.3; filter: blur(2px); } 

.map-background { display: none; }

.no-results { text-align: center; padding: 3rem; color: #ea580c; font-family: 'JetBrains Mono'; border: 1px dashed #333; margin: 2rem; }
.blink-text { animation: blink 1s step-end infinite; }

.pagination-controls { display: flex; justify-content: center; gap: 1rem; padding: 1.5rem 0; margin-top: auto; }
.page-btn {
  background: #111; border: 1px solid #333; color: #ea580c;
  font-family: 'JetBrains Mono'; padding: 8px 16px; cursor: pointer;
}
.page-btn:hover:not(:disabled) { border-color: #ea580c; background: rgba(234, 88, 12, 0.2); }
.page-btn:disabled { opacity: 0.3; cursor: not-allowed; }

.footer {
  margin-top: auto; border-top: 1px solid #222; background: #050505; padding: 0.8rem 1.5rem;
}
.footer-inner { display: flex; justify-content: space-between; max-width: 1200px; margin: 0 auto; }
.terminal-text { font-family: 'JetBrains Mono'; font-size: 0.7rem; color: #444; }

@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
.crt-fade-leave-active { transition: all 0.5s ease; }
.crt-fade-leave-to { opacity: 0; transform: scaleY(0.01) scaleX(1.5); filter: brightness(5); }
.zoom-fade-enter-active, .zoom-fade-leave-active { transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.zoom-fade-enter-from { opacity: 0; transform: scale(0.95) translateY(10px); }
.zoom-fade-leave-to { opacity: 0; transform: scale(1.05); filter: blur(4px); }
.slide-fade-enter-active { transition: all 0.3s ease-out; }
.slide-fade-leave-active { transition: all 0.2s cubic-bezier(1, 0.5, 0.8, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { transform: translateX(20px); opacity: 0; }

@media (min-width: 1024px) {
  .navbar { flex-direction: row; justify-content: space-between; }
  .brand { text-align: left; }
  .nav-controls { width: auto; gap: 1.5rem; }
  .search-container { width: 350px; flex: none; margin-right: 0; }
  
  .map-background {
    display: flex; position: fixed; top: 0; left: 0; width: 100%; height: 100%; 
    justify-content: center; opacity: 0.2; pointer-events: none; z-index: 0;
  }
  .world-map-img { width: 80%; filter: invert(1); mask-image: radial-gradient(circle, black 40%, transparent 80%); }
}
</style>