<script setup>
import { ref } from 'vue'

const albums = [
  { title: '慢慢懂', year: '2010', tracks: ['慢慢懂', '他的爱', '等不到你'], color: '#ff6b9d', icon: '🎹' },
  { title: '万有引力', year: '2012', tracks: ['有点甜', '万有引力', '风度'], color: '#ff8fb1', icon: '🎸' },
  { title: '传世乐章', year: '2014', tracks: ['传世乐章', '吵架歌', '不夜城'], color: '#ffb6c1', icon: '🎼' },
  { title: '登陆计划', year: '2016', tracks: ['登陆计划', '后会无期', '银河'], color: '#ff69b4', icon: '🎤' },
  { title: '克制凶猛', year: '2018', tracks: ['克制凶猛', '全城热恋', '怪咖'], color: '#e5547d', icon: '🥁' },
  { title: '大娱乐家', year: '2020', tracks: ['大娱乐家', '闪耀', '少年'], color: '#ff1493', icon: '🎷' },
  { title: '联名', year: '2022', tracks: ['联名', '年轮', '我想更懂你'], color: '#ff6b9d', icon: '🎺' },
  { title: '十万伏特', year: '2023', tracks: ['十万伏特', '小星星', '苦笑'], color: '#ff8fb1', icon: '🎻' },
  { title: '明日世界', year: '2026', tracks: ['明日世界', '待补充', '待补充'], color: '#ff6b9d', icon: '🪗' },
]

const flippedIndex = ref(-1)
const detailIndex = ref(-1)
</script>

<template>
  <section id="albums">
    <h2 class="section-title">专辑画廊</h2>
    <div class="album-grid">
      <div
        v-for="(album, index) in albums"
        :key="album.title"
        class="album-card"
        :class="{ flipped: flippedIndex === index }"
        @click="flippedIndex = flippedIndex === index ? -1 : index"
      >
        <div class="card-inner">
          <div class="card-front" :style="{ background: `linear-gradient(135deg, ${album.color}33, #0a0a1a)` }">
            <div class="album-cover" :style="{ borderColor: album.color }">
              <span class="album-icon">{{ album.icon }}</span>
            </div>
            <h3>{{ album.title }}</h3>
            <span class="year">{{ album.year }}</span>
          </div>
          <div class="card-back" :style="{ background: `linear-gradient(135deg, #0a0a1a, ${album.color}33)` }">
            <h3 :style="{ color: album.color }">{{ album.title }}</h3>
            <ul>
              <li v-for="track in album.tracks" :key="track">{{ track }}</li>
            </ul>
            <span class="hint">点击翻回</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
#albums {
  padding: 80px 40px;
  background: linear-gradient(180deg, #10081a 0%, #0a0a1a 100%);
}

.section-title {
  font-size: 36px;
  color: var(--pink);
  text-align: center;
  margin-bottom: 60px;
  letter-spacing: 4px;
}

.album-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 30px;
  max-width: 1000px;
  margin: 0 auto;
}

.album-card {
  perspective: 800px;
  cursor: pointer;
  height: 280px;
}

.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.album-card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 16px;
  border: 1px solid rgba(255, 107, 157, 0.2);
  padding: 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
}

.card-back {
  transform: rotateY(180deg);
}

.album-card:hover .card-inner {
  transform: rotateY(8deg) rotateX(-5deg);
}

.album-card.flipped:hover .card-inner {
  transform: rotateY(172deg) rotateX(-5deg);
}

.album-cover {
  width: 120px;
  height: 120px;
  border-radius: 12px;
  border: 2px solid;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  background: rgba(255, 255, 255, 0.05);
}

h3 {
  font-size: 18px;
  color: var(--text);
}

.year {
  font-size: 13px;
  color: var(--text-muted);
}

.card-back ul {
  list-style: none;
  padding: 0;
  text-align: center;
}

.card-back li {
  font-size: 14px;
  color: var(--text-muted);
  padding: 6px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.card-back li:last-child {
  border-bottom: none;
}

.hint {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: auto;
}
</style>
