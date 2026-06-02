<script setup>
import { ref, onMounted } from 'vue'

const defaultMessages = [
  { id: 1, text: '泷泷的歌陪伴了我整个青春！', color: '#ff6b9d', x: 10, y: 10, rotation: -3 },
  { id: 2, text: '从《有点甜》入坑，到现在每首歌都会唱', color: '#ff8fb1', x: 40, y: 5, rotation: 2 },
  { id: 3, text: '明日世界演唱会太震撼了！', color: '#ffb6c1', x: 70, y: 15, rotation: -1 },
  { id: 4, text: '小泷包永远支持你！', color: '#ff69b4', x: 20, y: 40, rotation: 3 },
  { id: 5, text: '期待更多新作品！', color: '#e5547d', x: 55, y: 35, rotation: -2 },
  { id: 6, text: '粉色海洋，永远爱你', color: '#ff1493', x: 80, y: 45, rotation: 1 },
]

const messages = ref([])
const newMsg = ref('')
const colors = ['#ff6b9d', '#ff8fb1', '#ffb6c1', '#ff69b4', '#e5547d', '#ff1493']

onMounted(() => {
  const saved = localStorage.getItem('wall-messages')
  messages.value = saved ? JSON.parse(saved) : [...defaultMessages]
})

function addMessage() {
  if (!newMsg.value.trim()) return
  messages.value.push({
    id: Date.now(),
    text: newMsg.value.trim(),
    color: colors[Math.floor(Math.random() * colors.length)],
    x: Math.random() * 70 + 5,
    y: Math.random() * 60 + 10,
    rotation: Math.random() * 8 - 4,
  })
  localStorage.setItem('wall-messages', JSON.stringify(messages.value))
  newMsg.value = ''
}

function removeMessage(id) {
  messages.value = messages.value.filter((m) => m.id !== id)
  localStorage.setItem('wall-messages', JSON.stringify(messages.value))
}
</script>

<template>
  <section id="message">
    <h2 class="section-title">留言墙</h2>
    <div class="add-bar">
      <input
        v-model="newMsg"
        placeholder="写下你想对泷泷说的话..."
        @keyup.enter="addMessage"
        maxlength="50"
      />
      <button @click="addMessage">贴上去</button>
    </div>
    <div class="wall">
      <div
        v-for="msg in messages"
        :key="msg.id"
        class="sticky"
        :style="{
          left: msg.x + '%',
          top: msg.y + '%',
          '--color': msg.color,
          transform: `rotate(${msg.rotation}deg)`,
        }"
        @dblclick="removeMessage(msg.id)"
      >
        <p>{{ msg.text }}</p>
        <span class="pin"></span>
      </div>
    </div>
    <p class="hint">双击便签可删除</p>
  </section>
</template>

<style scoped>
#message {
  padding: 80px 40px;
  background: linear-gradient(180deg, #0f0818 0%, #0a0a1a 100%);
  min-height: 100vh;
}

.section-title {
  font-size: 36px;
  color: var(--pink);
  text-align: center;
  margin-bottom: 30px;
  letter-spacing: 4px;
}

.add-bar {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-bottom: 40px;
}

.add-bar input {
  width: 320px;
  padding: 12px 20px;
  border-radius: 24px;
  border: 1px solid rgba(255, 107, 157, 0.3);
  background: rgba(255, 255, 255, 0.05);
  color: var(--text);
  font-size: 14px;
  outline: none;
  transition: border-color 0.3s;
}

.add-bar input:focus {
  border-color: var(--pink);
}

.add-bar button {
  padding: 12px 24px;
  border-radius: 24px;
  border: none;
  background: var(--pink);
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.add-bar button:hover {
  background: var(--pink-dark);
  box-shadow: 0 0 15px var(--pink-glow);
}

.wall {
  position: relative;
  width: 100%;
  height: 500px;
  max-width: 1000px;
  margin: 0 auto;
}

.sticky {
  position: absolute;
  width: 160px;
  min-height: 100px;
  padding: 24px 16px 16px;
  background: var(--color);
  border-radius: 2px 2px 4px 4px;
  cursor: grab;
  transition: transform 0.2s, box-shadow 0.2s;
  animation: sway 3s ease-in-out infinite;
}

.sticky:hover {
  transform: scale(1.05) !important;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
  z-index: 10;
}

.sticky p {
  font-size: 13px;
  color: #1a1a1a;
  line-height: 1.6;
}

.pin {
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #c0392b;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

@keyframes sway {
  0%, 100% { transform: rotate(var(--rotate, 0deg)); }
  50% { transform: rotate(calc(var(--rotate, 0deg) + 1deg)); }
}

.hint {
  text-align: center;
  color: var(--text-muted);
  font-size: 12px;
  margin-top: 20px;
}

@media (max-width: 768px) {
  .add-bar {
    flex-direction: column;
    align-items: center;
  }

  .add-bar input {
    width: 100%;
    max-width: 320px;
  }

  .sticky {
    width: 130px;
    min-height: 80px;
    padding: 20px 12px 12px;
    font-size: 12px;
  }
}
</style>
