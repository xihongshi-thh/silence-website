<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const lyrics = [
  { text: '我愿意为你被放逐天际', song: '有点甜', color: '#ff6b9d' },
  { text: '不分手的恋爱', song: '不分手的恋爱', color: '#ff8fb1' },
  { text: '一闪一闪亮晶晶', song: '小星星', color: '#ffb6c1' },
  { text: '你是我最美的风景', song: '苦笑', color: '#ff69b4' },
  { text: '全世界都安静', song: '风度', color: '#ff1493' },
  { text: '想把你写成一首歌', song: '那个男孩', color: '#ff6b9d' },
  { text: '年轮增加了几圈', song: '年轮', color: '#e5547d' },
  { text: '后会无期的约定', song: '后会无期', color: '#ff8fb1' },
  { text: '我想更懂你', song: '我想更懂你', color: '#ff1493' },
  { text: '慢慢懂了', song: '慢慢懂', color: '#ffb6c1' },
  { text: '你是我的独家记忆', song: '独家记忆', color: '#ff69b4' },
  { text: '万有引力', song: '万有引力', color: '#e5547d' },
]

const cards = ref([])
const favorites = ref(JSON.parse(localStorage.getItem('lyrics-fav') || '[]'))
let animId = null

onMounted(() => {
  cards.value = lyrics.map((l, i) => ({
    ...l,
    id: i,
    x: Math.random() * 80 + 10,
    y: -20 - Math.random() * 100,
    speed: 0.05 + Math.random() * 0.1,
    rotation: Math.random() * 20 - 10,
    flipped: false,
    paused: false,
  }))

  function animate() {
    cards.value.forEach((c) => {
      if (!c.paused) {
        c.y += c.speed
        if (c.y > 110) {
          c.y = -20
          c.x = Math.random() * 80 + 10
        }
      }
    })
    animId = requestAnimationFrame(animate)
  }
  animate()
})

onUnmounted(() => {
  if (animId) cancelAnimationFrame(animId)
})

function toggleFav(id) {
  const idx = favorites.value.indexOf(id)
  if (idx >= 0) {
    favorites.value.splice(idx, 1)
  } else {
    favorites.value.push(id)
  }
  localStorage.setItem('lyrics-fav', JSON.stringify(favorites.value))
}
</script>

<template>
  <section id="lyrics">
    <h2 class="section-title">歌词墙</h2>
    <div class="lyrics-container">
      <div
        v-for="card in cards"
        :key="card.id"
        class="lyric-card"
        :class="{ flipped: card.flipped }"
        :style="{
          left: card.x + '%',
          top: card.y + '%',
          transform: `rotate(${card.rotation}deg)`,
          borderColor: card.color,
        }"
        @mouseenter="card.paused = true"
        @mouseleave="card.paused = false"
        @click="card.flipped = !card.flipped"
      >
        <div class="card-inner">
          <div class="card-front">
            <p class="lyric-text">{{ card.text }}</p>
            <span class="fav-btn" :class="{ active: favorites.includes(card.id) }" @click.stop="toggleFav(card.id)">
              {{ favorites.includes(card.id) ? '❤️' : '🤍' }}
            </span>
          </div>
          <div class="card-back">
            <p class="song-name">{{ card.song }}</p>
            <p class="hint">点击翻回</p>
          </div>
        </div>
      </div>
    </div>
    <div class="favorites-bar" v-if="favorites.length > 0">
      <span class="fav-label">我的最爱：</span>
      <span v-for="fid in favorites" :key="fid" class="fav-tag">
        {{ lyrics[fid].song }}
      </span>
    </div>
  </section>
</template>

<style scoped>
#lyrics {
  padding: 80px 40px;
  min-height: 100vh;
  background: linear-gradient(180deg, #0a0a1a 0%, #10081a 100%);
  position: relative;
  overflow: hidden;
}

.section-title {
  font-size: 36px;
  color: var(--pink);
  text-align: center;
  margin-bottom: 40px;
  letter-spacing: 4px;
  position: relative;
  z-index: 2;
}

.lyrics-container {
  position: relative;
  width: 100%;
  height: 60vh;
}

.lyric-card {
  position: absolute;
  width: 180px;
  height: 100px;
  perspective: 600px;
  cursor: pointer;
  z-index: 1;
  transition: z-index 0s;
}

.lyric-card:hover {
  z-index: 10;
}

.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.5s;
}

.lyric-card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 12px;
  border: 1px solid;
  padding: 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.card-front {
  background: rgba(10, 10, 26, 0.9);
  backdrop-filter: blur(10px);
}

.card-back {
  background: rgba(255, 107, 157, 0.15);
  transform: rotateY(180deg);
}

.lyric-card:hover .card-front {
  box-shadow: 0 0 20px var(--pink-glow);
}

.lyric-text {
  font-size: 14px;
  text-align: center;
  line-height: 1.6;
  color: var(--text);
}

.fav-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  font-size: 14px;
  cursor: pointer;
}

.song-name {
  font-size: 14px;
  color: var(--pink);
  font-weight: 600;
}

.hint {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: 8px;
}

.favorites-bar {
  position: relative;
  z-index: 2;
  margin-top: 40px;
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  justify-content: center;
}

.fav-label {
  font-size: 14px;
  color: var(--text-muted);
}

.fav-tag {
  font-size: 12px;
  padding: 4px 12px;
  border-radius: 20px;
  background: rgba(255, 107, 157, 0.2);
  border: 1px solid var(--pink);
  color: var(--pink);
}
</style>
