<script setup>
import { computed } from 'vue';

const props = defineProps({
  time: Date,
  timezone: String, 
  city: String,
  offset: String
});

const clockData = computed(() => {
  const validTime = props.time || new Date();
  
  const formatter = new Intl.DateTimeFormat('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false,
    timeZone: props.timezone 
  });

  const parts = formatter.formatToParts(validTime);
  const h = parseInt(parts.find(p => p.type === 'hour').value);
  const m = parseInt(parts.find(p => p.type === 'minute').value);
  const s = parseInt(parts.find(p => p.type === 'second').value);
  const hProgress = ((h % 12) + m / 60) / 12 * 100;
  const mProgress = (m / 60) * 100;
  
  return {
    hProgress,
    mProgress,
    sRotation: s * 6, // 360 / 60
    digital: `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}`,
    isNight: h < 6 || h >= 18
  };
});
</script>

<template>
  <div class="clock-node" :class="{ 'night-mode': clockData.isNight }">
    <div class="corner tl"></div>
    <div class="corner tr"></div>
    <div class="corner bl"></div>
    <div class="corner br"></div>
    
    <div class="node-header">
      <div class="status-dot"></div>
      <span class="city">{{ city }}</span>
      <span class="offset">{{ offset }}</span>
    </div>

    <div class="visual-engine">
      <svg viewBox="0 0 100 100" class="main-svg">
        <circle cx="50" cy="50" r="40" class="ring-bg" />
        <circle cx="50" cy="50" r="28" class="ring-bg" />
        
        <circle cx="50" cy="50" r="40" 
          class="ring-active minute-ring"
          pathLength="100"
          stroke-dasharray="100"
          :stroke-dashoffset="100 - clockData.mProgress" 
        />
        
        <circle cx="50" cy="50" r="28" 
          class="ring-active hour-ring"
          pathLength="100"
          stroke-dasharray="100"
          :stroke-dashoffset="100 - clockData.hProgress" 
        />

        <line x1="50" y1="50" x2="50" y2="10" 
          class="scanner-line"
          :style="{ transform: `rotate(${clockData.sRotation}deg)` }"
        />
      </svg>
      
      <div class="center-content">
        <div class="time-display">{{ clockData.digital }}</div>
        <div class="label">TIME</div>
      </div>
    </div>
    
    <div class="node-footer">
      <div class="signal-bar">
        <div class="bar active"></div>
        <div class="bar active"></div>
        <div class="bar" :class="{ 'active': !clockData.isNight }"></div>
      </div>
      <span class="mode-text">{{ clockData.isNight ? 'NIGHT OPS' : 'DAY OPS' }}</span>
    </div>
  </div>
</template>

<style scoped>
.clock-node {
  --primary: #f97316; 
  --primary-dim: rgba(249, 115, 22, 0.2);
  --primary-glow: rgba(249, 115, 22, 0.4);
  --bg-card: linear-gradient(145deg, #0a0a0a, #050505);
  
  position: relative;
  width: 100%;
  height: 100%;
  aspect-ratio: 1 / 1.15; 
  padding: 1rem;
  box-sizing: border-box;
  
  background: var(--bg-card);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  transition: all 0.3s ease;
}

.night-mode {
  --primary: #06b6d4;
  --primary-dim: rgba(6, 182, 212, 0.2);
  --primary-glow: rgba(6, 182, 212, 0.4);
}

.clock-node:hover {
  border-color: var(--primary);
  box-shadow: 0 4px 20px rgba(0,0,0,0.5);
  transform: translateY(-2px);
}

.corner {
  position: absolute; width: 6px; height: 6px;
  border-color: var(--primary); border-style: solid; opacity: 0.5;
  transition: opacity 0.3s;
}
.clock-node:hover .corner { opacity: 1; }
.tl { top: 0; left: 0; border-width: 2px 0 0 2px; border-radius: 4px 0 0 0; }
.tr { top: 0; right: 0; border-width: 2px 2px 0 0; border-radius: 0 4px 0 0; }
.bl { bottom: 0; left: 0; border-width: 0 0 2px 2px; border-radius: 0 0 0 4px; }
.br { bottom: 0; right: 0; border-width: 0 2px 2px 0; border-radius: 0 0 4px 0; }

.node-header {
  display: flex; align-items: center; gap: 8px;
  margin-bottom: 0.5rem;
}

.status-dot {
  width: 6px; height: 6px; background: var(--primary);
  border-radius: 50%; box-shadow: 0 0 6px var(--primary);
  animation: pulse 3s infinite;
}

.city { 
  font-size: 0.85rem; font-weight: 700; color: #fff; letter-spacing: 1px; 
  text-transform: uppercase;
}
.offset { 
  font-size: 0.7rem; color: #666; margin-left: auto; font-weight: 600; 
}

.visual-engine {
  flex: 1; 
  position: relative; 
  display: flex; justify-content: center; align-items: center;
  width: 100%;
}

.main-svg {
  width: 80%; height: auto; 
  max-width: 140px;
  transform: rotate(-90deg); 
}

.ring-bg { 
  fill: none; stroke: #1a1a1a; stroke-width: 6; 
}

.ring-active { 
  fill: none; stroke-width: 6; stroke-linecap: round; 
  stroke: var(--primary);
  transition: stroke-dashoffset 1s linear; 
  filter: drop-shadow(0 0 2px var(--primary));
}

.scanner-line {
  stroke: var(--primary); stroke-width: 1.5; stroke-opacity: 0.8;
  transform-origin: center;
  transition: transform 1s linear;
}

.center-content {
  position: absolute; text-align: center;
  top: 50%; left: 50%; transform: translate(-50%, -50%);
}

.time-display {
  font-family: 'Inter', sans-serif;
  font-size: 1.4rem; font-weight: 600; color: #fff;
  line-height: 1; margin-bottom: 2px;
  text-shadow: 0 2px 4px rgba(0,0,0,0.5);
}

.label { 
  font-size: 0.5rem; color: #555; letter-spacing: 1.5px; font-weight: 700;
}

.node-footer {
  margin-top: auto; display: flex; justify-content: space-between; align-items: flex-end;
  padding-top: 0.8rem; border-top: 1px solid rgba(255,255,255,0.05);
}

.signal-bar { display: flex; gap: 3px; align-items: flex-end; height: 10px; }
.bar { width: 3px; height: 40%; background: #333; border-radius: 1px; }
.bar.active { background: var(--primary); box-shadow: 0 0 4px var(--primary-glow); }
.bar:nth-child(2) { height: 70%; }
.bar:nth-child(3) { height: 100%; }

.mode-text { 
  font-size: 0.65rem; color: #444; font-weight: 800; letter-spacing: 0.5px; 
}

@keyframes pulse {
  0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; }
}

@media (max-width: 768px) {
  .clock-node { padding: 0.8rem; }
  .time-display { font-size: 1.1rem; }
  .city { font-size: 0.75rem; }
  .main-svg { width: 75%; }
}
</style>