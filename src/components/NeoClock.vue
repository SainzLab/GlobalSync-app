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
    hour: '2-digit', minute: '2-digit', second: '2-digit',
    hour12: false,
    timeZone: props.timezone 
  });

  const parts = formatter.formatToParts(validTime);
  const h = parseInt(parts.find(p => p.type === 'hour').value);
  const m = parseInt(parts.find(p => p.type === 'minute').value);
  const s = parseInt(parts.find(p => p.type === 'second').value);
  const hProgress = ((h % 12) + m / 60) / 12 * 100;
  const mProgress = (m / 60) * 100;
  const isNight = h < 6 || h >= 18;

  return {
    hProgress,
    mProgress,
    sRotation: s * 6, 
    digital: `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}`,
    isNight,
    statusText: isNight ? 'NIGHT' : 'DAY'
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
      <div class="header-left">
        <div class="status-dot"></div>
        <span class="city">{{ city }}</span>
      </div>
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
        <div class="bar active"></div> <div class="bar active"></div> <div class="bar" :class="{ 'active': !clockData.isNight }"></div> 
      </div>
      <span class="mode-text">{{ clockData.statusText }}</span>
    </div>

  </div>
</template>

<style scoped>
.clock-node {
  --primary: #f97316; 
  --primary-dim: rgba(249, 115, 22, 0.1);
  --bg-gradient: linear-gradient(145deg, #0a0a0a 0%, #050505 100%);
  --text-color: #ffffff;
  
  position: relative;
  width: 100%;
  height: 100%;
  aspect-ratio: 1 / 1.15; 
  padding: 1.2rem;
  box-sizing: border-box;
  
  background: var(--bg-gradient);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 6px; 
  overflow: hidden;
  
  display: flex;
  flex-direction: column;
  transition: all 0.3s ease;
}

.night-mode {
  --primary: #06b6d4; 
  --primary-dim: rgba(6, 182, 212, 0.1);
}

.clock-node:hover {
  border-color: var(--primary);
  box-shadow: 0 10px 40px -10px rgba(0,0,0,0.8);
  transform: translateY(-2px);
}

.corner {
  position: absolute; width: 8px; height: 8px;
  border-color: var(--primary); border-style: solid;
  opacity: 0.7; transition: opacity 0.3s;
}
.tl { top: 0; left: 0; border-width: 2px 0 0 2px; border-top-left-radius: 6px; }
.tr { top: 0; right: 0; border-width: 2px 2px 0 0; border-top-right-radius: 6px; }
.bl { bottom: 0; left: 0; border-width: 0 0 2px 2px; border-bottom-left-radius: 6px; }
.br { bottom: 0; right: 0; border-width: 0 2px 2px 0; border-bottom-right-radius: 6px; }

.clock-node:hover .corner { opacity: 1; box-shadow: 0 0 5px var(--primary); }

.node-header {
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 0.5rem;
}
.header-left { display: flex; align-items: center; gap: 8px; }

.status-dot {
  width: 6px; height: 6px; border-radius: 50%;
  background: var(--primary);
  box-shadow: 0 0 6px var(--primary);
  animation: pulse 3s infinite;
}

.city { 
  font-family: 'Inter', sans-serif; 
  font-size: 0.85rem; font-weight: 800; 
  letter-spacing: 1px; text-transform: uppercase; color: #fff;
}

.offset { 
  font-size: 0.75rem; color: #666; font-weight: 600; 
}

.visual-engine {
  flex: 1; position: relative; 
  display: flex; justify-content: center; align-items: center;
  width: 100%;
}

.main-svg {
  width: 80%; height: auto; max-width: 160px;
  transform: rotate(-90deg); 
}

.ring-bg { 
  fill: none; stroke: #151515; stroke-width: 6; 
}

.ring-active { 
  fill: none; stroke-width: 6; stroke-linecap: round; 
  stroke: var(--primary);
  transition: stroke-dashoffset 1s linear; 
  filter: drop-shadow(0 0 2px var(--primary));
}

.scanner-line {
  stroke: var(--primary); stroke-width: 1.5; stroke-opacity: 0.6;
  transform-origin: center; transition: transform 1s linear;
}

.center-content {
  position: absolute; text-align: center;
  top: 50%; left: 50%; transform: translate(-50%, -50%);
}

.time-display {
  font-family: 'Inter', sans-serif;
  font-size: 1.6rem; font-weight: 500; color: #fff;
  line-height: 1; letter-spacing: 1px;
}

.label { 
  font-size: 0.5rem; color: #555; letter-spacing: 2px; 
  font-weight: 700; margin-top: 4px;
}

.node-footer {
  margin-top: auto; padding-top: 10px;
  border-top: 1px solid rgba(255,255,255,0.05);
  display: flex; justify-content: space-between; align-items: flex-end;
}

.signal-bar { display: flex; gap: 3px; align-items: flex-end; height: 10px; }
.bar { width: 3px; height: 30%; background: #333; border-radius: 1px; }
.bar.active { background: var(--primary); box-shadow: 0 0 4px var(--primary); }
.bar:nth-child(2) { height: 60%; }
.bar:nth-child(3) { height: 100%; }

.mode-text { 
  font-size: 0.65rem; color: #555; font-weight: 800; letter-spacing: 0.5px; 
}

@keyframes pulse {
  0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; }
}

@media (max-width: 768px) {
  .clock-node { padding: 1rem; }
  .time-display { font-size: 1.4rem; }
  .city { font-size: 0.8rem; }
}
</style>