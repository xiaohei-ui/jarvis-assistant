<script setup>
import { ref, onMounted } from 'vue'

const showOverlay = ref(true)
const logContent = ref('')
const voiceBarStyle = ref({ transform: 'translateX(-100%)' })
const logs = ref([])

function log(message) {
  const time = new Date().toLocaleTimeString()
  logs.value.unshift(`[${time}] ${message}`)
  if (logs.value.length > 20) {
    logs.value.pop()
  }
  logContent.value = logs.value.join('\n')
}

function speak(text, callback) {
  if (!window.speechSynthesis) {
    log('Speech synthesis not supported')
    if (callback) callback()
    return
  }

  const utterance = new SpeechSynthesisUtterance(text)
  utterance.rate = 1
  utterance.pitch = 1

  const voices = window.speechSynthesis.getVoices()
  const englishVoice = voices.find(v => v.lang.includes('en'))
  if (englishVoice) {
    utterance.voice = englishVoice
  }

  voiceBarStyle.value = { animation: 'pulse-bar 1s ease-in-out infinite' }

  utterance.onend = () => {
    voiceBarStyle.value = { transform: 'translateX(-100%)' }
    if (callback) callback()
  }

  log(`JARVIS: ${text}`)
  window.speechSynthesis.speak(utterance)
}

function initializeSystem() {
  showOverlay.value = false
  log('System initialized')
  log('Loading HUD components...')
  log('Connecting to mainframe...')
  log('Welcome, Wu Wei.')

  setTimeout(() => {
    speak('Hello Wu Wei. All systems are online. How may I assist you today?')
  }, 1000)
}

onMounted(() => {
  // Preload voices
  if (window.speechSynthesis) {
    window.speechSynthesis.getVoices()
  }
})
</script>

<template>
  <div class="jarvis-container">
    <div class="grid-bg"></div>

    <div v-if="showOverlay" id="init-overlay" @click="initializeSystem">
      <div class="init-btn">INITIALIZE SYSTEM</div>
    </div>

    <div class="hud">
      <div class="header-left box">J.A.R.V.I.S. MK IV</div>
      <div class="header-right box">
        <span>WU WEI EDITION</span>
      </div>

      <div class="sidebar-left">
        <div class="box">
          <div class="box-title">SYSTEM STATUS</div>
          <div class="status-item">
            <span>POWER</span>
            <span class="status-value active">ONLINE</span>
          </div>
          <div class="status-item">
            <span>NETWORK</span>
            <span class="status-value active">CONNECTED</span>
          </div>
          <div class="status-item">
            <span>AI CORE</span>
            <span class="status-value active">ACTIVE</span>
          </div>
        </div>
        <div class="box">
          <div class="box-title">DIAGNOSTICS</div>
          <div class="diag-bar">
            <div class="diag-fill" style="width: 85%"></div>
          </div>
          <div class="diag-label">CPU: 85%</div>
          <div class="diag-bar">
            <div class="diag-fill" style="width: 60%"></div>
          </div>
          <div class="diag-label">MEMORY: 60%</div>
        </div>
      </div>

      <div class="center-stage">
        <div class="scale-ring sr2"></div>
        <div class="scale-ring sr1"></div>
        <div class="atom-sphere">
          <div class="orbit o1"></div>
          <div class="orbit o2"></div>
          <div class="orbit o3"></div>
          <div class="core"></div>
        </div>
      </div>

      <div class="sidebar-right">
        <div class="box">
          <div class="box-title">ALERTS</div>
          <div class="alert-item">No threats detected</div>
        </div>
        <div class="box">
          <div class="box-title">MODULES</div>
          <div class="module-item">
            <span class="module-indicator active"></span>
            <span>Voice Interface</span>
          </div>
          <div class="module-item">
            <span class="module-indicator active"></span>
            <span>HUD Display</span>
          </div>
          <div class="module-item">
            <span class="module-indicator"></span>
            <span>Combat Mode</span>
          </div>
        </div>
      </div>

      <div class="footer">
        <div class="box log-panel">
          <pre>{{ logContent }}</pre>
        </div>
        <div class="voice-status">
          <div class="voice-bar" :style="voiceBarStyle"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
:root {
  --cyan: #00f3ff;
  --dark-cyan: rgba(0, 243, 255, 0.2);
  --bg: #050505;
}

.jarvis-container {
  margin: 0;
  overflow: hidden;
  background: var(--bg);
  font-family: 'Rajdhani', sans-serif;
  color: var(--cyan);
  perspective: 1000px;
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}

body {
  margin: 0;
  padding: 0;
}

.grid-bg {
  position: absolute;
  width: 200%;
  height: 200%;
  top: -50%;
  left: -50%;
  background:
    linear-gradient(transparent 0%, rgba(0, 243, 255, 0.05) 1%, transparent 2%),
    linear-gradient(90deg, transparent 0%, rgba(0, 243, 255, 0.05) 1%, transparent 2%);
  background-size: 100px 100px;
  transform: rotateX(60deg) translateY(100px);
  animation: grid-move 20s linear infinite;
  z-index: -1;
}

.hud {
  position: relative;
  width: 100vw;
  height: 100vh;
  display: grid;
  grid-template-columns: 300px 1fr 300px;
  grid-template-rows: 100px 1fr 150px;
  pointer-events: none;
}

.box {
  border: 1px solid var(--dark-cyan);
  background: rgba(0, 20, 30, 0.3);
  padding: 15px;
  position: relative;
  backdrop-filter: blur(2px);
}

.box::before {
  content: '';
  position: absolute;
  top: -1px;
  left: -1px;
  width: 10px;
  height: 10px;
  border-top: 2px solid var(--cyan);
  border-left: 2px solid var(--cyan);
}

.box::after {
  content: '';
  position: absolute;
  bottom: -1px;
  right: -1px;
  width: 10px;
  height: 10px;
  border-bottom: 2px solid var(--cyan);
  border-right: 2px solid var(--cyan);
}

.box-title {
  font-size: 12px;
  letter-spacing: 3px;
  margin-bottom: 10px;
  opacity: 0.7;
}

.header-left {
  grid-column: 1;
  grid-row: 1;
  display: flex;
  align-items: center;
  padding-left: 20px;
  font-size: 1.5em;
  letter-spacing: 5px;
  text-shadow: 0 0 10px var(--cyan);
}

.header-right {
  grid-column: 3;
  grid-row: 1;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding-right: 20px;
}

.sidebar-left {
  grid-column: 1;
  grid-row: 2;
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
}

.sidebar-right {
  grid-column: 3;
  grid-row: 2;
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
}

.footer {
  grid-column: 1 / 4;
  grid-row: 3;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.center-stage {
  grid-column: 2;
  grid-row: 2;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  transform-style: preserve-3d;
}

.atom-sphere {
  position: relative;
  width: 200px;
  height: 200px;
  transform-style: preserve-3d;
  animation: rotate-sphere 10s linear infinite;
}

.orbit {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100%;
  height: 100%;
  border: 1px solid var(--cyan);
  border-radius: 50%;
  box-shadow: 0 0 10px var(--cyan);
}

.o1 {
  transform: translate(-50%, -50%) rotateY(0deg);
}

.o2 {
  transform: translate(-50%, -50%) rotateY(60deg);
}

.o3 {
  transform: translate(-50%, -50%) rotateY(120deg);
}

.core {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 40px;
  height: 40px;
  background: var(--cyan);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 50px var(--cyan), 0 0 100px white;
  animation: pulse-core 2s ease-in-out infinite alternate;
}

.scale-ring {
  position: absolute;
  border: 1px dashed var(--dark-cyan);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.sr1 {
  width: 400px;
  height: 400px;
  border-width: 2px;
  animation: spin 20s linear infinite;
}

.sr2 {
  width: 550px;
  height: 550px;
  border-left: 10px solid var(--cyan);
  border-right: 10px solid var(--cyan);
  opacity: 0.3;
  animation: spin-rev 15s linear infinite;
}

.log-panel {
  height: 150px;
  width: 80%;
  max-width: 800px;
  overflow: hidden;
  font-size: 12px;
  font-family: 'Courier New', monospace;
  opacity: 0.8;
  mask-image: linear-gradient(to bottom, black 80%, transparent 100%);
}

.log-panel pre {
  margin: 0;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.voice-status {
  width: 300px;
  height: 4px;
  background: #333;
  margin-top: 10px;
  position: relative;
  overflow: hidden;
}

.voice-bar {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  background: var(--cyan);
  box-shadow: 0 0 20px var(--cyan);
  transition: transform 0.1s;
}

#init-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  z-index: 999;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  pointer-events: auto;
}

.init-btn {
  padding: 20px 60px;
  border: 2px solid var(--cyan);
  color: var(--cyan);
  font-size: 24px;
  letter-spacing: 8px;
  text-transform: uppercase;
  background: transparent;
  box-shadow: 0 0 30px var(--cyan);
  transition: 0.3s;
}

.init-btn:hover {
  background: var(--cyan);
  color: #000;
}

/* Status Items */
.status-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 14px;
}

.status-value {
  color: #666;
}

.status-value.active {
  color: #00ff00;
}

/* Diagnostics */
.diag-bar {
  height: 8px;
  background: #222;
  margin-bottom: 5px;
  border-radius: 4px;
  overflow: hidden;
}

.diag-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--cyan), #00ff00);
}

.diag-label {
  font-size: 11px;
  margin-bottom: 10px;
  opacity: 0.7;
}

/* Alerts */
.alert-item {
  font-size: 12px;
  color: #00ff00;
}

/* Modules */
.module-item {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 8px;
  font-size: 12px;
}

.module-indicator {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #666;
}

.module-indicator.active {
  background: #00ff00;
  box-shadow: 0 0 10px #00ff00;
}

/* Keyframes */
@keyframes grid-move {
  0% {
    transform: rotateX(60deg) translateY(0);
  }
  100% {
    transform: rotateX(60deg) translateY(100px);
  }
}

@keyframes rotate-sphere {
  0% {
    transform: rotateX(0) rotateY(0);
  }
  100% {
    transform: rotateX(360deg) rotateY(360deg);
  }
}

@keyframes spin {
  100% {
    transform: rotate(360deg);
  }
}

@keyframes spin-rev {
  100% {
    transform: rotate(-360deg);
  }
}

@keyframes pulse-core {
  0% {
    opacity: 0.5;
    transform: translate(-50%, -50%) scale(0.8);
  }
  100% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1.2);
  }
}

@keyframes pulse-bar {
  0% {
    transform: translateX(-100%);
  }
  50% {
    transform: translateX(0%);
  }
  100% {
    transform: translateX(100%);
  }
}
</style>
