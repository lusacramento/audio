<template>
  <div class="microphone-container">
    <h1>Real-Time Microphone Frequency Analyzer</h1>

    <div class="controls">
      <button @click="start" :disabled="isRecording" class="control-btn start-btn">
        🎤 Start Listening
      </button>
      <button @click="stop" :disabled="!isRecording" class="control-btn stop-btn">
        ⏹️ Stop
      </button>
    </div>

    <div class="visualization">
      <div class="frequency-display">
        <div class="value-box">
          <span class="label">Frequency</span>
          <span class="value">{{ frequency }} Hz</span>
        </div>
        <div class="value-box">
          <span class="label">Volume</span>
          <span class="value">{{ volume }} dB</span>
        </div>
      </div>

      <div class="volume-meter">
        <div class="meter-fill" :style="{ width: volumePercentage + '%' }" :class="{
          'low': volumePercentage < 33,
          'medium': volumePercentage >= 33 && volumePercentage < 66,
          'high': volumePercentage >= 66
        }"></div>
      </div>

      <div class="frequency-bars">
        <div v-for="(bar, index) in frequencyBars" :key="index" class="frequency-bar" :style="{ height: bar + '%' }">
        </div>
      </div>
    </div>

    <div v-if="error" class="error-message">
      ❌ {{ error }}
    </div>

    <div class="status">
      <div class="status-indicator" :class="{ active: isRecording }"></div>
      Status: {{ isRecording ? 'Listening...' : 'Stopped' }}
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onUnmounted, computed } from 'vue'
import { UserMedia, Analyser } from 'tone'

const frequency = ref(0)
const volume = ref(-100)
const isRecording = ref(false)
const error = ref('')
const frequencyBars = ref(Array(32).fill(0))

let mic: UserMedia | null = null
let analyser: Analyser | null = null
let animationFrameId: number | null = null

const volumePercentage = computed(() => {
  // Convert dB (-100 to 0) to percentage (0 to 100)
  return Math.max(0, Math.min(100, (volume.value + 100)))
})

async function start() {
  if (isRecording.value) return

  try {
    error.value = ''

    // Create analyser with proper configuration
    analyser = new Analyser('fft', 2048)

    // Create and open microphone
    mic = new UserMedia()
    mic.connect(analyser)

    await mic.open()
    isRecording.value = true

    // Start analysis
    analyzeWithTone()

  } catch (err) {
    console.error('Error with Tone.js:', err)
    error.value = 'Failed to access microphone. Please check permissions and try again.'
    isRecording.value = false
  }
}

function analyzeWithTone() {
  if (!analyser || !isRecording.value) return

  try {
    // Get frequency data
    const values = analyser.getValue() as Float32Array

    if (values && values.length > 0) {
      // Calculate volume (RMS)
      let sum = 0
      for (let i = 0; i < values.length; i++) {
        sum += Math.abs(values[i])
      }
      const average = sum / values.length
      volume.value = Math.round(average * 1000) // Scale for better visualization

      // Update frequency bars for visualization
      updateFrequencyBars(values)

      // Find peak frequency
      let maxIndex = 0
      let maxValue = 0
      for (let i = 0; i < values.length; i++) {
        if (Math.abs(values[i]) > maxValue) {
          maxValue = Math.abs(values[i])
          maxIndex = i
        }
      }

      // Convert to frequency (approximate)
      frequency.value = Math.round(maxIndex * (44100 / 2) / values.length)
    }

    animationFrameId = requestAnimationFrame(analyzeWithTone)
  } catch (err) {
    console.error('Analysis error:', err)
    stop()
  }
}

function updateFrequencyBars(values: Float32Array) {
  const bars = 32
  const samplesPerBar = Math.floor(values.length / bars)

  for (let i = 0; i < bars; i++) {
    let sum = 0
    for (let j = 0; j < samplesPerBar; j++) {
      const index = i * samplesPerBar + j
      if (index < values.length) {
        sum += Math.abs(values[index])
      }
    }
    const average = sum / samplesPerBar
    // Scale for visualization (0-100%)
    frequencyBars.value[i] = Math.min(100, average * 500)
  }
}

function stop() {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
    animationFrameId = null
  }

  if (mic) {
    mic.close()
    mic.dispose()
    mic = null
  }

  if (analyser) {
    analyser.dispose()
    analyser = null
  }

  isRecording.value = false
  // Reset visualization
  frequencyBars.value = Array(32).fill(0)
}

onUnmounted(() => {
  stop()
})
</script>

<style scoped>
.microphone-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: white;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
  font-size: 2rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.controls {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 30px;
}

.control-btn {
  padding: 15px 30px;
  border: none;
  border-radius: 25px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.control-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.start-btn {
  background: linear-gradient(45deg, #4CAF50, #45a049);
  color: white;
}

.start-btn:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.stop-btn {
  background: linear-gradient(45deg, #f44336, #da190b);
  color: white;
}

.stop-btn:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.visualization {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 30px;
  margin-bottom: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.frequency-display {
  display: flex;
  justify-content: space-around;
  margin-bottom: 30px;
}

.value-box {
  text-align: center;
  background: rgba(255, 255, 255, 0.2);
  padding: 20px;
  border-radius: 15px;
  min-width: 150px;
}

.label {
  display: block;
  font-size: 0.9rem;
  opacity: 0.8;
  margin-bottom: 5px;
}

.value {
  display: block;
  font-size: 2rem;
  font-weight: bold;
}

.volume-meter {
  width: 100%;
  height: 30px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  overflow: hidden;
  margin-bottom: 30px;
}

.meter-fill {
  height: 100%;
  transition: width 0.1s ease;
  border-radius: 15px;
}

.meter-fill.low {
  background: linear-gradient(90deg, #4CAF50, #8BC34A);
}

.meter-fill.medium {
  background: linear-gradient(90deg, #FFC107, #FF9800);
}

.meter-fill.high {
  background: linear-gradient(90deg, #FF5722, #f44336);
}

.frequency-bars {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  height: 150px;
  gap: 2px;
}

.frequency-bar {
  flex: 1;
  background: linear-gradient(to top, #00bcd4, #3f51b5);
  border-radius: 3px 3px 0 0;
  transition: height 0.1s ease;
  min-height: 2px;
}

.error-message {
  background: rgba(244, 67, 54, 0.2);
  border: 1px solid rgba(244, 67, 54, 0.5);
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 20px;
  text-align: center;
}

.status {
  text-align: center;
  font-size: 1.1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.status-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #ccc;
  transition: background 0.3s ease;
}

.status-indicator.active {
  background: #4CAF50;
  box-shadow: 0 0 10px #4CAF50;
}

@media (max-width: 768px) {
  .controls {
    flex-direction: column;
    align-items: center;
  }

  .frequency-display {
    flex-direction: column;
    gap: 15px;
  }

  .value-box {
    min-width: auto;
  }
}
</style>