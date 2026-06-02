<script setup>
import { ref, onMounted } from 'vue'

const flipped = ref(false)
const countUp = ref({ years: 0, albums: 0, songs: 0 })
const sectionVisible = ref(false)

const info = {
  name: '汪苏泷',
  english: 'Silence Wang',
  birthday: '1989年9月17日',
  birthplace: '辽宁省沈阳市',
  zodiac: '处女座',
  education: '沈阳音乐学院',
  fans: '小泷包',
  color: '粉色',
}

const quote = '音乐是我和这个世界对话的方式。'

onMounted(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting && !sectionVisible.value) {
        sectionVisible.value = true
        animateCount()
      }
    },
    { threshold: 0.3 }
  )
  observer.observe(document.getElementById('about'))
})

function animateCount() {
  const targets = { years: 16, albums: 9, songs: 100 }
  const duration = 2000
  const start = Date.now()

  function update() {
    const elapsed = Date.now() - start
    const progress = Math.min(elapsed / duration, 1)
    const ease = 1 - Math.pow(1 - progress, 3)

    countUp.value.years = Math.floor(targets.years * ease)
    countUp.value.albums = Math.floor(targets.albums * ease)
    countUp.value.songs = Math.floor(targets.songs * ease)

    if (progress < 1) requestAnimationFrame(update)
  }
  update()
}
</script>

<template>
  <section id="about">
    <h2 class="section-title">关于他</h2>
    <div class="about-container" :class="{ visible: sectionVisible }">
      <div class="card-wrapper" @click="flipped = !flipped">
        <div class="card" :class="{ flipped }">
          <div class="card-front">
            <img src="/images/avatar.jpg" alt="汪苏泷" class="avatar-img" />
            <p class="flip-hint">点击翻转</p>
          </div>
          <div class="card-back">
            <p class="quote">"{{ quote }}"</p>
            <p class="flip-hint">点击翻回</p>
          </div>
        </div>
      </div>

      <div class="info-panel">
        <div class="info-grid">
          <div class="info-item" v-for="(value, key) in info" :key="key">
            <span class="label">{{ key === 'name' ? '姓名' : key === 'english' ? '英文名' : key === 'birthday' ? '生日' : key === 'birthplace' ? '出生地' : key === 'zodiac' ? '星座' : key === 'education' ? '毕业院校' : key === 'fans' ? '粉丝名' : '应援色' }}</span>
            <span class="value">{{ value }}</span>
          </div>
        </div>

        <div class="stats">
          <div class="stat">
            <span class="stat-num">{{ countUp.years }}</span>
            <span class="stat-label">出道年份</span>
          </div>
          <div class="stat">
            <span class="stat-num">{{ countUp.albums }}</span>
            <span class="stat-label">专辑数</span>
          </div>
          <div class="stat">
            <span class="stat-num">{{ countUp.songs }}+</span>
            <span class="stat-label">原创歌曲</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
#about {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 40px;
  background: linear-gradient(180deg, #0a0a1a 0%, #0f0a1a 100%);
}

.section-title {
  font-size: 36px;
  color: var(--pink);
  margin-bottom: 60px;
  letter-spacing: 4px;
}

.about-container {
  display: flex;
  gap: 60px;
  max-width: 1000px;
  width: 100%;
  align-items: center;
  opacity: 0;
  transform: translateY(40px);
  transition: all 0.8s ease;
}

.about-container.visible {
  opacity: 1;
  transform: translateY(0);
}

.card-wrapper {
  perspective: 1000px;
  cursor: pointer;
  flex-shrink: 0;
}

.card {
  width: 260px;
  height: 340px;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.card.flipped {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 30px;
}

.card-front {
  background: linear-gradient(135deg, var(--pink-dark), #2a1a2a);
  border: 1px solid rgba(255, 107, 157, 0.3);
}

.card-back {
  background: linear-gradient(135deg, #2a1a2a, var(--pink-dark));
  border: 1px solid rgba(255, 107, 157, 0.3);
  transform: rotateY(180deg);
}

.avatar-img {
  width: 180px;
  height: 180px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid var(--pink);
  box-shadow: 0 0 20px var(--pink-glow);
  margin-top: 30px;
  margin-bottom: 20px;
}

.quote {
  font-size: 16px;
  line-height: 1.8;
  color: var(--text);
  text-align: center;
  font-style: italic;
}

.flip-hint {
  font-size: 12px;
  color: var(--text-muted);
  margin-top: 16px;
}

.info-panel {
  flex: 1;
}

.info-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 40px;
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.label {
  font-size: 12px;
  color: var(--text-muted);
  letter-spacing: 1px;
}

.value {
  font-size: 16px;
  color: var(--text);
}

.stats {
  display: flex;
  gap: 40px;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.stat-num {
  font-size: 36px;
  font-weight: 700;
  color: var(--pink);
}

.stat-label {
  font-size: 12px;
  color: var(--text-muted);
  letter-spacing: 1px;
}

@media (max-width: 768px) {
  .about-container {
    flex-direction: column;
    gap: 40px;
  }

  .card {
    width: 220px;
    height: 280px;
  }

  .info-grid {
    grid-template-columns: 1fr;
  }

  .stats {
    justify-content: center;
  }
}
</style>
