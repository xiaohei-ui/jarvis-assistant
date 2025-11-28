<template>
  <div class="jarvis-container" @click="initSystem">
    <!-- Click overlay for initialization -->
    <div v-if="!systemInitialized" class="init-overlay">
      <div class="init-text">CLICK TO INITIALIZE J.A.R.V.I.S. MK IV</div>
    </div>

    <!-- Three Column Layout -->
    <div class="main-grid">
      <!-- Left Panel - System Status -->
      <div class="panel left-panel">
        <div class="panel-header">SYSTEM STATUS</div>
        <div class="status-item">
          <span class="status-label">POWER</span>
          <span class="status-value" :class="{ active: systemInitialized }">{{ systemInitialized ? 'ONLINE' : 'STANDBY' }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">VOICE</span>
          <span class="status-value" :class="{ active: voiceReady }">{{ voiceReady ? 'READY' : 'LOADING' }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">SPEECH REC</span>
          <span class="status-value" :class="{ active: isListening }">{{ isListening ? 'ACTIVE' : 'IDLE' }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">AI CORE</span>
          <span class="status-value active">OPERATIONAL</span>
        </div>
      </div>

      <!-- Center Panel - Arc Reactor -->
      <div class="panel center-panel">
        <div class="reactor-container">
          <div class="reactor-core" :class="{ active: systemInitialized, speaking: isSpeaking }">
            <div class="reactor-ring ring-1"></div>
            <div class="reactor-ring ring-2"></div>
            <div class="reactor-ring ring-3"></div>
            <div class="reactor-center"></div>
          </div>
        </div>
        <div class="jarvis-label">J.A.R.V.I.S.</div>
        <div class="version-label">MK IV INTERFACE</div>
        
        <!-- Voice Controls -->
        <div class="voice-controls" v-if="systemInitialized">
          <button 
            class="voice-btn" 
            :class="{ listening: isListening }"
            @click.stop="toggleListening"
          >
            {{ isListening ? 'LISTENING...' : 'SPEAK' }}
          </button>
        </div>
        
        <!-- Current Speech Display -->
        <div class="speech-display" v-if="currentSpeech">
          <div class="speech-text">"{{ currentSpeech }}"</div>
        </div>
      </div>

      <!-- Right Panel - Log Panel -->
      <div class="panel right-panel">
        <div class="panel-header">ACTIVITY LOG</div>
        <div class="log-container" ref="logContainer">
          <div 
            v-for="(log, index) in logs" 
            :key="index" 
            class="log-entry"
            :class="log.type"
          >
            <span class="log-time">{{ log.time }}</span>
            <span class="log-message">{{ log.message }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Bottom Status Bar -->
    <div class="status-bar">
      <div class="status-bar-item">STARK INDUSTRIES</div>
      <div class="status-bar-item">{{ currentTime }}</div>
      <div class="status-bar-item">SECURE CONNECTION</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      systemInitialized: false,
      voiceReady: false,
      isListening: false,
      isSpeaking: false,
      currentSpeech: '',
      currentTime: '',
      logs: [],
      recognition: null,
      synthesis: window.speechSynthesis,
      selectedVoice: null,
      voicesLoaded: false,
    }
  },
  mounted() {
    this.updateTime()
    setInterval(this.updateTime, 1000)
    this.loadVoices()
  },
  methods: {
    updateTime() {
      const now = new Date()
      this.currentTime = now.toLocaleTimeString('en-US', { 
        hour12: false, 
        hour: '2-digit', 
        minute: '2-digit', 
        second: '2-digit' 
      })
    },
    
    addLog(message, type = 'info') {
      const now = new Date()
      const time = now.toLocaleTimeString('en-US', { 
        hour12: false, 
        hour: '2-digit', 
        minute: '2-digit', 
        second: '2-digit' 
      })
      this.logs.push({ time, message, type })
      
      // Auto scroll to bottom
      this.$nextTick(() => {
        if (this.$refs.logContainer) {
          this.$refs.logContainer.scrollTop = this.$refs.logContainer.scrollHeight
        }
      })
    },
    
    loadVoices() {
      const loadVoiceList = () => {
        const voices = this.synthesis.getVoices()
        if (voices.length > 0) {
          // Try to find a good English voice
          this.selectedVoice = voices.find(voice => 
            voice.name.includes('Google UK English Male')
          ) || voices.find(voice => 
            voice.name.includes('English') && voice.name.includes('Male')
          ) || voices.find(voice => 
            voice.lang.startsWith('en')
          ) || voices[0]
          
          this.voicesLoaded = true
          this.addLog(`Voice loaded: ${this.selectedVoice?.name || 'default'}`, 'system')
        }
      }
      
      // Load voices immediately if available
      loadVoiceList()
      
      // Also listen for voiceschanged event (required for some browsers)
      if (this.synthesis.onvoiceschanged !== undefined) {
        this.synthesis.onvoiceschanged = loadVoiceList
      }
    },
    
    async initSystem() {
      if (this.systemInitialized) return
      
      this.addLog('Initializing J.A.R.V.I.S. MK IV...', 'system')
      
      // Resume audio context and unlock speech synthesis
      try {
        // Resume speech synthesis (required after user gesture)
        this.synthesis.resume()
        
        // Play a silent utterance to unlock the audio context
        const silentUtterance = new SpeechSynthesisUtterance('')
        silentUtterance.volume = 0
        this.synthesis.speak(silentUtterance)
        
        // Wait a moment for audio context to unlock
        await new Promise(resolve => setTimeout(resolve, 100))
        
        this.addLog('Audio context unlocked', 'success')
      } catch (e) {
        this.addLog('Audio context warning: ' + e.message, 'warning')
      }
      
      // Initialize speech recognition
      if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
        this.recognition = new SpeechRecognition()
        this.recognition.continuous = false
        this.recognition.interimResults = false
        this.recognition.lang = 'en-US'
        
        this.recognition.onresult = (event) => {
          const transcript = event.results[0][0].transcript
          this.addLog(`User: "${transcript}"`, 'user')
          this.processCommand(transcript)
        }
        
        this.recognition.onerror = (event) => {
          this.addLog(`Speech recognition error: ${event.error}`, 'error')
          this.isListening = false
        }
        
        this.recognition.onend = () => {
          this.isListening = false
        }
        
        this.addLog('Speech recognition initialized', 'success')
      } else {
        this.addLog('Speech recognition not supported', 'error')
      }
      
      this.systemInitialized = true
      this.voiceReady = true
      
      // Welcome message with a slight delay to ensure voices are ready
      setTimeout(() => {
        this.speak('J.A.R.V.I.S. MK IV online. All systems operational, sir.')
      }, 500)
    },
    
    speak(text) {
      if (!this.synthesis) {
        this.addLog('Speech synthesis not available', 'error')
        return
      }
      
      // Cancel any ongoing speech
      this.synthesis.cancel()
      
      // Resume synthesis (required for some browsers)
      this.synthesis.resume()
      
      const utterance = new SpeechSynthesisUtterance(text)
      
      // Use selected voice if available
      if (this.selectedVoice) {
        utterance.voice = this.selectedVoice
      }
      
      utterance.rate = 1.0
      utterance.pitch = 1.0
      utterance.volume = 1.0
      
      utterance.onstart = () => {
        this.isSpeaking = true
        this.currentSpeech = text
        this.addLog(`JARVIS: "${text}"`, 'jarvis')
      }
      
      utterance.onend = () => {
        this.isSpeaking = false
        this.currentSpeech = ''
      }
      
      utterance.onerror = (event) => {
        this.isSpeaking = false
        this.currentSpeech = ''
        this.addLog(`Speech error: ${event.error}`, 'error')
      }
      
      this.synthesis.speak(utterance)
    },
    
    toggleListening() {
      if (this.isListening) {
        this.recognition?.stop()
        this.isListening = false
      } else {
        try {
          this.recognition?.start()
          this.isListening = true
          this.addLog('Listening for command...', 'system')
        } catch (e) {
          this.addLog(`Recognition error: ${e.message}`, 'error')
        }
      }
    },
    
    processCommand(command) {
      const lowerCommand = command.toLowerCase()
      
      if (lowerCommand.includes('hello') || lowerCommand.includes('hi')) {
        this.speak('Hello, sir. How may I assist you today?')
      } else if (lowerCommand.includes('time')) {
        const now = new Date()
        const timeStr = now.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit', hour12: true })
        this.speak(`The current time is ${timeStr}.`)
      } else if (lowerCommand.includes('status')) {
        this.speak('All systems are functioning within normal parameters, sir.')
      } else if (lowerCommand.includes('weather')) {
        this.speak('I apologize, sir. Weather data integration is not yet configured.')
      } else if (lowerCommand.includes('jarvis')) {
        this.speak('Yes, sir. I am here and ready to assist.')
      } else {
        this.speak(`I understand you said: ${command}. How may I help with that?`)
      }
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #0a0a0f;
  font-family: 'Segoe UI', 'Roboto', sans-serif;
  overflow: hidden;
}

.jarvis-container {
  width: 100vw;
  height: 100vh;
  background: linear-gradient(135deg, #0a0a0f 0%, #1a1a2e 50%, #0a0a0f 100%);
  position: relative;
  display: flex;
  flex-direction: column;
}

/* Init Overlay */
.init-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  cursor: pointer;
}

.init-text {
  color: #00d4ff;
  font-size: 24px;
  font-weight: 300;
  letter-spacing: 4px;
  text-shadow: 0 0 20px #00d4ff, 0 0 40px #00d4ff;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}

/* Main Grid - Three Column Layout */
.main-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 20px;
  padding: 20px;
  flex: 1;
  min-height: 0;
}

/* Panel Styles */
.panel {
  background: rgba(0, 100, 150, 0.1);
  border: 1px solid rgba(0, 212, 255, 0.3);
  border-radius: 10px;
  padding: 20px;
  position: relative;
  overflow: hidden;
}

.panel::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #00d4ff, transparent);
}

.panel-header {
  color: #00d4ff;
  font-size: 14px;
  letter-spacing: 3px;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 1px solid rgba(0, 212, 255, 0.2);
}

/* Left Panel - Status */
.left-panel {
  display: flex;
  flex-direction: column;
}

.status-item {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid rgba(0, 212, 255, 0.1);
}

.status-label {
  color: #6a6a8a;
  font-size: 12px;
  letter-spacing: 2px;
}

.status-value {
  color: #ff4444;
  font-size: 12px;
  letter-spacing: 1px;
}

.status-value.active {
  color: #00ff88;
  text-shadow: 0 0 10px #00ff88;
}

/* Center Panel - Reactor */
.center-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.reactor-container {
  position: relative;
  width: 250px;
  height: 250px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.reactor-core {
  position: relative;
  width: 200px;
  height: 200px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.reactor-ring {
  position: absolute;
  border-radius: 50%;
  border: 2px solid rgba(0, 212, 255, 0.3);
}

.ring-1 {
  width: 100%;
  height: 100%;
  animation: rotate 20s linear infinite;
}

.ring-2 {
  width: 75%;
  height: 75%;
  animation: rotate 15s linear infinite reverse;
}

.ring-3 {
  width: 50%;
  height: 50%;
  animation: rotate 10s linear infinite;
}

.reactor-core.active .reactor-ring {
  border-color: rgba(0, 212, 255, 0.8);
  box-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
}

.reactor-core.speaking .reactor-ring {
  border-color: rgba(0, 255, 136, 0.8);
  box-shadow: 0 0 30px rgba(0, 255, 136, 0.7);
}

.reactor-center {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: radial-gradient(circle, #00d4ff 0%, #0066aa 50%, #003366 100%);
  box-shadow: 0 0 30px #00d4ff, 0 0 60px #00d4ff, inset 0 0 20px rgba(255, 255, 255, 0.3);
}

.reactor-core.active .reactor-center {
  box-shadow: 0 0 40px #00d4ff, 0 0 80px #00d4ff, 0 0 120px #00d4ff, inset 0 0 30px rgba(255, 255, 255, 0.5);
}

.reactor-core.speaking .reactor-center {
  background: radial-gradient(circle, #00ff88 0%, #00aa66 50%, #006644 100%);
  box-shadow: 0 0 40px #00ff88, 0 0 80px #00ff88, 0 0 120px #00ff88;
  animation: pulse-glow 0.5s ease-in-out infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes pulse-glow {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

.jarvis-label {
  color: #00d4ff;
  font-size: 36px;
  font-weight: 200;
  letter-spacing: 15px;
  margin-top: 30px;
  text-shadow: 0 0 20px #00d4ff;
}

.version-label {
  color: #6a6a8a;
  font-size: 12px;
  letter-spacing: 5px;
  margin-top: 10px;
}

/* Voice Controls */
.voice-controls {
  margin-top: 30px;
}

.voice-btn {
  background: transparent;
  border: 2px solid #00d4ff;
  color: #00d4ff;
  padding: 15px 40px;
  font-size: 14px;
  letter-spacing: 3px;
  cursor: pointer;
  transition: all 0.3s ease;
  border-radius: 5px;
}

.voice-btn:hover {
  background: rgba(0, 212, 255, 0.2);
  box-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
}

.voice-btn.listening {
  background: rgba(0, 255, 136, 0.2);
  border-color: #00ff88;
  color: #00ff88;
  animation: pulse 1s ease-in-out infinite;
}

/* Speech Display */
.speech-display {
  margin-top: 20px;
  padding: 15px 25px;
  background: rgba(0, 212, 255, 0.1);
  border: 1px solid rgba(0, 212, 255, 0.3);
  border-radius: 5px;
  max-width: 400px;
}

.speech-text {
  color: #00d4ff;
  font-size: 14px;
  font-style: italic;
  text-align: center;
}

/* Right Panel - Logs */
.right-panel {
  display: flex;
  flex-direction: column;
}

.log-container {
  flex: 1;
  overflow-y: auto;
  padding-right: 10px;
}

.log-container::-webkit-scrollbar {
  width: 4px;
}

.log-container::-webkit-scrollbar-thumb {
  background: #00d4ff;
  border-radius: 2px;
}

.log-entry {
  padding: 8px 0;
  border-bottom: 1px solid rgba(0, 212, 255, 0.1);
  font-size: 11px;
}

.log-time {
  color: #4a4a6a;
  margin-right: 10px;
  font-family: monospace;
}

.log-message {
  color: #8a8aaa;
}

.log-entry.system .log-message {
  color: #00d4ff;
}

.log-entry.success .log-message {
  color: #00ff88;
}

.log-entry.error .log-message {
  color: #ff4444;
}

.log-entry.warning .log-message {
  color: #ffaa00;
}

.log-entry.user .log-message {
  color: #ff88ff;
}

.log-entry.jarvis .log-message {
  color: #00d4ff;
  font-weight: 500;
}

/* Status Bar */
.status-bar {
  display: flex;
  justify-content: space-between;
  padding: 15px 30px;
  background: rgba(0, 50, 100, 0.3);
  border-top: 1px solid rgba(0, 212, 255, 0.3);
}

.status-bar-item {
  color: #4a4a6a;
  font-size: 11px;
  letter-spacing: 2px;
}

/* Responsive */
@media (max-width: 1024px) {
  .main-grid {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr auto;
  }
  
  .left-panel {
    order: 2;
  }
  
  .center-panel {
    order: 1;
  }
  
  .right-panel {
    order: 3;
    max-height: 200px;
  }
}
</style>
