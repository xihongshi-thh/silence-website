<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const playlist = [
  { title: '有点甜', duration: '4:12', file: 'youdiantian.mp3' },
  { title: '不分手的恋爱', duration: '4:35', file: 'bufenshoudeailian.mp3' },
  { title: '小星星', duration: '3:48', file: 'xiaoxingxing.mp3' },
  { title: '苦笑', duration: '4:01', file: 'kuxiao.mp3' },
  { title: '风度', duration: '4:22', file: 'fengdu.mp3' },
  { title: '那个男孩', duration: '3:56', file: 'nagenanhai.mp3' },
  { title: '年轮', duration: '4:44', file: 'nianlun.mp3' },
  { title: '后会无期', duration: '4:18', file: 'houhuiwuqi.mp3' },
]

const currentTrack = ref(0)
const isPlaying = ref(false)
const progress = ref(0)
const showList = ref(false)
const bars = ref([])
const volume = ref(0.5)
const showVolume = ref(false)
const bgmEnabled = ref(true)

let audio = null
let audioCtx = null
let analyser = null
let source = null
let bgmOscillators = []
let animId = null
let bgmGain = null

// BGM melody notes (simple piano-like ambient)
const melodyNotes = [
  { freq: 523.25, dur: 0.8 }, // C5
  { freq: 587.33, dur: 0.8 }, // D5
  { freq: 659.25, dur: 1.2 }, // E5
  { freq: 523.25, dur: 0.8 }, // C5
  { freq: 659.25, dur: 0.8 }, // E5
  { freq: 587.33, dur: 1.2 }, // D5
  { freq: 523.25, dur: 0.8 }, // C5
  { freq: 493.88, dur: 0.8 }, // B4
  { freq: 440.00, dur: 1.2 }, // A4
  { freq: 493.88, dur: 0.8 }, // B4
  { freq: 523.25, dur: 1.2 }, // C5
]

function initAudio() {
  if (audioCtx) return
  audioCtx = new (window.AudioContext || window.webkitAudioContext)()
  analyser = audioCtx.createAnalyser()
  analyser.fftSize = 128
  bgmGain = audioCtx.createGain()
  bgmGain.gain.value = volume.value * 0.15
  bgmGain.connect(analyser)
  analyser.connect(audioCtx.destination)
}

function startBGM() {
  if (!bgmEnabled.value || !audioCtx) return
  stopBGM()

  let time = audioCtx.currentTime

  function playMelodyLoop() {
    melodyNotes.forEach((note) => {
      const osc = audioCtx.createOscillator()
      const gain = audioCtx.createGain()

      osc.type = 'sine'
      osc.frequency.value = note.freq

      gain.gain.setValueAtTime(0, time)
      gain.gain.linearRampToValueAtTime(0.3, time + 0.05)
      gain.gain.exponentialRampToValueAtTime(0.01, time + note.dur)

      osc.connect(gain)
      gain.connect(bgmGain)

      osc.start(time)
      osc.stop(time + note.dur)

      bgmOscillators.push(osc)
      time += note.dur
    })
  }

  playMelodyLoop()

  // Loop the melody
  const loopInterval = setInterval(() => {
    if (!bgmEnabled.value || !isPlaying.value) {
      clearInterval(loopInterval)
      return
    }
    playMelodyLoop()
  }, melodyNotes.reduce((sum, n) => sum + n.dur, 0) * 1000)

  bgmOscillators._loopInterval = loopInterval
}

function stopBGM() {
  bgmOscillators.forEach((osc) => {
    try { osc.stop() } catch (e) {}
  })
  if (bgmOscillators._loopInterval) {
    clearInterval(bgmOscillators._loopInterval)
  }
  bgmOscillators = []
}

function loadTrack(index) {
  if (audio) {
    audio.pause()
    audio.src = ''
  }

  const track = playlist[index]
  audio = new Audio(`/music/${track.file}`)
  audio.volume = volume.value
  audio.loop = false

  audio.addEventListener('ended', () => {
    nextTrack()
  })

  audio.addEventListener('timeupdate', () => {
    if (audio.duration) {
      progress.value = (audio.currentTime / audio.duration) * 100
    }
  })

  audio.addEventListener('error', () => {
    // If mp3 not found, just update progress visually
    console.log(`Audio file not found: ${track.file}. Place mp3 files in public/music/`)
  })
}

onMounted(() => {
  bars.value = Array.from({ length: 40 }, () => Math.random() * 0.8 + 0.2)
  loadTrack(0)

  function animateBars() {
    if (isPlaying.value && analyser) {
      const dataArray = new Uint8Array(analyser.frequencyBinCount)
      analyser.getByteFrequencyData(dataArray)
      bars.value = Array.from(dataArray).slice(0, 40).map((v) => v / 255)
    } else if (isPlaying.value) {
      bars.value = bars.value.map(() => Math.random() * 0.8 + 0.2)
    }
    animId = requestAnimationFrame(animateBars)
  }
  animateBars()
})

onUnmounted(() => {
  if (animId) cancelAnimationFrame(animId)
  if (audio) audio.pause()
  stopBGM()
  if (audioCtx) audioCtx.close()
})

watch(volume, (v) => {
  if (audio) audio.volume = v
  if (bgmGain) bgmGain.gain.value = v * 0.15
})

function togglePlay() {
  initAudio()

  if (audioCtx.state === 'suspended') {
    audioCtx.resume()
  }

  isPlaying.value = !isPlaying.value

  if (isPlaying.value) {
    audio.play().catch(() => {})
    if (bgmEnabled.value) startBGM()
  } else {
    audio.pause()
    stopBGM()
  }
}

function nextTrack() {
  currentTrack.value = (currentTrack.value + 1) % playlist.length
  loadTrack(currentTrack.value)
  progress.value = 0
  if (isPlaying.value) {
    audio.play().catch(() => {})
  }
}

function prevTrack() {
  currentTrack.value = (currentTrack.value - 1 + playlist.length) % playlist.length
  loadTrack(currentTrack.value)
  progress.value = 0
  if (isPlaying.value) {
    audio.play().catch(() => {})
  }
}

function selectTrack(index) {
  currentTrack.value = index
  loadTrack(index)
  progress.value = 0
  initAudio()
  if (audioCtx.state === 'suspended') audioCtx.resume()
  isPlaying.value = true
  audio.play().catch(() => {})
  if (bgmEnabled.value) startBGM()
  showList.value = false
}

function seekTo(e) {
  if (!audio || !audio.duration) return
  const rect = e.currentTarget.getBoundingClientRect()
  const percent = (e.clientX - rect.left) / rect.width
  audio.currentTime = percent * audio.duration
}

function toggleBgm() {
  bgmEnabled.value = !bgmEnabled.value
  if (bgmEnabled.value && isPlaying.value) {
    startBGM()
  } else {
    stopBGM()
  }
}
</script>

<template>
  <div class="player" :class="{ expanded: showList }">
    <!-- Waveform visualization -->
    <div class="waveform">
      <div
        v-for="(bar, i) in bars"
        :key="i"
        class="bar"
        :style="{
          height: isPlaying ? Math.max(bar * 100, 5) + '%' : '10%',
        }"
      ></div>
    </div>

    <!-- Progress bar -->
    <div class="progress-bar" @click="seekTo">
      <div class="progress-fill" :style="{ width: progress + '%' }"></div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button class="ctrl-btn" @click="prevTrack" title="上一首">⏮</button>
      <button class="play-btn" @click="togglePlay" :title="isPlaying ? '暂停' : '播放'">
        {{ isPlaying ? '⏸' : '▶' }}
      </button>
      <button class="ctrl-btn" @click="nextTrack" title="下一首">⏭</button>

      <div class="track-info">
        <span class="track-title">{{ playlist[currentTrack].title }}</span>
        <span class="track-duration">{{ playlist[currentTrack].duration }}</span>
      </div>

      <!-- Volume -->
      <div class="volume-wrap" @mouseenter="showVolume = true" @mouseleave="showVolume = false">
        <button class="ctrl-btn" title="音量">
          {{ volume > 0.5 ? '🔊' : volume > 0 ? '🔉' : '🔇' }}
        </button>
        <div v-show="showVolume" class="volume-slider">
          <input type="range" min="0" max="1" step="0.05" v-model.number="volume" />
        </div>
      </div>

      <!-- BGM toggle -->
      <button class="ctrl-btn bgm-btn" :class="{ active: bgmEnabled }" @click="toggleBgm" title="BGM 伴奏">
        ♪
      </button>

      <button class="list-btn" @click="showList = !showList" title="播放列表">☰</button>
    </div>

    <!-- Playlist -->
    <div v-show="showList" class="playlist">
      <div
        v-for="(track, index) in playlist"
        :key="track.title"
        class="playlist-item"
        :class="{ active: index === currentTrack }"
        @click="selectTrack(index)"
      >
        <span class="track-num">{{ index + 1 }}</span>
        <span class="track-name">{{ track.title }}</span>
        <span class="track-time">{{ track.duration }}</span>
      </div>
      <div class="playlist-hint">
        将 mp3 文件放入 <code>public/music/</code> 目录即可播放
      </div>
    </div>
  </div>
</template>

<style scoped>
.player {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(10, 10, 26, 0.95);
  backdrop-filter: blur(20px);
  border-top: 1px solid rgba(255, 107, 157, 0.2);
  z-index: 1000;
  transition: all 0.3s;
}

.waveform {
  display: flex;
  align-items: flex-end;
  justify-content: center;
  height: 30px;
  gap: 2px;
  padding: 8px 20px 0;
}

.bar {
  width: 3px;
  background: var(--pink);
  border-radius: 2px;
  transition: height 0.12s ease;
  min-height: 3px;
  opacity: 0.8;
}

.progress-bar {
  height: 4px;
  background: rgba(255, 255, 255, 0.1);
  margin: 4px 20px;
  border-radius: 2px;
  cursor: pointer;
  position: relative;
}

.progress-bar:hover {
  height: 6px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--pink-dark), var(--pink));
  border-radius: 2px;
  transition: width 0.1s linear;
}

.controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 8px 20px 12px;
}

.ctrl-btn,
.play-btn,
.list-btn {
  background: none;
  border: none;
  color: var(--text);
  cursor: pointer;
  font-size: 18px;
  padding: 4px 8px;
  transition: color 0.2s;
}

.ctrl-btn:hover,
.list-btn:hover {
  color: var(--pink);
}

.play-btn {
  width: 42px;
  height: 42px;
  border-radius: 50%;
  background: var(--pink);
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  color: white;
}

.play-btn:hover {
  background: var(--pink-dark);
  box-shadow: 0 0 15px var(--pink-glow);
  transform: scale(1.05);
}

.track-info {
  display: flex;
  flex-direction: column;
  min-width: 120px;
}

.track-title {
  font-size: 14px;
  color: var(--text);
}

.track-duration {
  font-size: 11px;
  color: var(--text-muted);
}

.volume-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.volume-slider {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(10, 10, 26, 0.95);
  border: 1px solid rgba(255, 107, 157, 0.2);
  border-radius: 8px;
  padding: 12px 8px;
}

.volume-slider input[type="range"] {
  writing-mode: vertical-lr;
  direction: rtl;
  width: 20px;
  height: 80px;
  appearance: none;
  background: transparent;
}

.volume-slider input[type="range"]::-webkit-slider-track {
  width: 4px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 2px;
}

.volume-slider input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: var(--pink);
  cursor: pointer;
}

.bgm-btn {
  font-size: 20px;
  font-weight: bold;
  opacity: 0.4;
  transition: all 0.2s;
}

.bgm-btn.active {
  opacity: 1;
  color: var(--pink);
  text-shadow: 0 0 8px var(--pink-glow);
}

.playlist {
  max-height: 280px;
  overflow-y: auto;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.playlist-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 24px;
  cursor: pointer;
  transition: background 0.2s;
}

.playlist-item:hover {
  background: rgba(255, 107, 157, 0.1);
}

.playlist-item.active {
  background: rgba(255, 107, 157, 0.15);
}

.track-num {
  font-size: 12px;
  color: var(--text-muted);
  width: 20px;
}

.track-name {
  flex: 1;
  font-size: 14px;
  color: var(--text);
}

.playlist-item.active .track-name {
  color: var(--pink);
}

.track-time {
  font-size: 12px;
  color: var(--text-muted);
}

.playlist-hint {
  text-align: center;
  padding: 12px;
  font-size: 12px;
  color: var(--text-muted);
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.playlist-hint code {
  background: rgba(255, 107, 157, 0.15);
  padding: 2px 6px;
  border-radius: 4px;
  color: var(--pink);
  font-size: 11px;
}
</style>
