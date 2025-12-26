<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import DashboardView from './components/DashboardView.vue';
import CalendarView from './components/CalendarView.vue';

const currentTime = ref(new Date());
const isCalendarOpen = ref(false); 
const searchQuery = ref('');
let timer = null;

onMounted(() => { timer = setInterval(() => currentTime.value = new Date(), 1000); });
onUnmounted(() => clearInterval(timer));
</script>

<template>
  <div class="app-container">
    
    <header class="navbar">
      <div class="brand">
        <h1>GLOBAL<span class="accent">SYNC</span></h1>
      </div>
      
      <div class="nav-controls">
        <transition name="fade">
          <div v-if="!isCalendarOpen" class="search-container">
            <input 
              type="text" 
              v-model="searchQuery" 
              placeholder="> Search City" 
              class="terminal-input"
            />
          </div>
        </transition>

        <button class="cyber-toggle" :class="{ 'active': isCalendarOpen }" @click="isCalendarOpen = !isCalendarOpen">
          <span class="toggle-label">VIEW</span>
          <div class="toggle-switch">
            <span class="state-text">{{ isCalendarOpen ? 'Calender' : 'dashboard' }}</span>
          </div>
        </button>
      </div>
    </header>

    <div class="map-background">
      <img src="/world-map.svg" class="world-map-img" />
      <div class="fade-overlay"></div>
    </div>

    <div class="content-area">
      <transition name="fade" mode="out-in">
        <component 
          :is="isCalendarOpen ? CalendarView : DashboardView"
          :searchQuery="searchQuery"
          :currentTime="currentTime"
        />
      </transition>
    </div>

  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&family=JetBrains+Mono:wght@400;700&display=swap');
body { margin: 0; background: #000; color: white; font-family: 'Inter', sans-serif; }
</style>

<style scoped>
.app-container { min-height: 100vh; display: flex; flex-direction: column; }

.navbar {
  position: sticky; top: 0; z-index: 50; 
  background: rgba(10, 10, 10, 0.95);
  border-bottom: 1px solid rgba(234, 88, 12, 0.2);
  padding: 1rem 1.5rem;
  display: flex; flex-direction: column; gap: 1rem;
  backdrop-filter: blur(10px);
}

.brand { text-align: center; }
h1 { 
  font-size: 1.5rem; margin: 0; font-weight: 800; color: #e5e5e5; letter-spacing: 2px; 
  text-shadow: 2px 0 rgba(234, 88, 12, 0.2);
}
.accent { color: #ea580c; }

.nav-controls {
  display: flex; 
  width: 100%;
  align-items: center;
  min-height: 42px; 
  position: relative;
}

.search-container {
  flex: 1; 
  margin-right: 15px; 
  max-width: 100%;
}

.terminal-input {
  width: 100%;
  background: rgba(0, 0, 0, 0.5);
  border: 1px solid #333;
  border-left: 3px solid #666;
  color: #ea580c;
  padding: 10px 12px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.85rem;
  outline: none;
  transition: all 0.3s ease;
  box-sizing: border-box; 
}
.terminal-input:focus {
  border-color: #ea580c;
  background: rgba(234, 88, 12, 0.05);
}

.cyber-toggle {
  margin-left: auto; 
  
  display: flex;
  align-items: center;
  gap: 10px;
  background: transparent;
  border: 1px solid #333;
  padding: 4px 6px 4px 12px; 
  border-radius: 4px;
  cursor: pointer;
  font-family: 'JetBrains Mono', monospace;
  transition: all 0.3s ease;
}

.toggle-label {
  font-size: 0.7rem;
  color: #666;
  font-weight: 700;
  letter-spacing: 1px;
}

.toggle-switch {
  background: #111;
  border: 1px solid #444;
  padding: 6px 12px;
  border-radius: 2px;
  color: #ea580c;
  font-weight: bold;
  font-size: 0.8rem;
  min-width: 40px; 
  text-align: center;
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  box-shadow: 0 0 10px rgba(0,0,0,0.5) inset;
}

.cyber-toggle:hover { border-color: #666; }
.cyber-toggle:hover .toggle-switch {
  background: #ea580c;
  color: #000;
  box-shadow: 0 0 15px rgba(234, 88, 12, 0.4);
}

.cyber-toggle.active { border-color: #ea580c; }
.cyber-toggle.active .toggle-label { color: #ea580c; }
.cyber-toggle.active .toggle-switch {
  background: #ea580c;
  color: black;
  border-color: #ea580c;
  box-shadow: 0 0 10px rgba(234, 88, 12, 0.5);
}

.map-background { display: none; }
.content-area { flex: 1; position: relative; z-index: 10; padding-top: 1rem; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

@media (min-width: 1024px) {
  .navbar { flex-direction: row; justify-content: space-between; align-items: center; padding: 1.5rem 3rem; }
  .brand { text-align: left; }
  .nav-controls { width: auto; gap: 1.5rem; min-height: auto; }
  .search-container { width: 300px; flex: none; margin-right: 0; }
  .cyber-toggle { margin-left: 0; }
  
  .map-background {
    display: flex; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    justify-content: center; opacity: 0.3; pointer-events: none; z-index: 0;
  }
  .world-map-img { width: 80%; filter: invert(0.2); mask-image: linear-gradient(to bottom, black 30%, transparent); }
}
</style>