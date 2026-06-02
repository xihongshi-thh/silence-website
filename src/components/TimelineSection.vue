<script setup>
import { ref } from 'vue'

const timeline = [
  { year: '2010', title: '出道', desc: '发布首张专辑《慢慢懂》，以网络音乐人身份出道', icon: '🎵' },
  { year: '2012', title: '《万有引力》', desc: '发行第二张专辑，收录《有点甜》等热门歌曲', icon: '💫' },
  { year: '2014', title: '《传世乐章》', desc: '音乐风格逐渐成熟，展现多元化创作能力', icon: '🎼' },
  { year: '2016', title: '《登陆计划》', desc: '首次个人巡回演唱会启动', icon: '🚀' },
  { year: '2018', title: '《克制凶猛》', desc: '音乐风格突破，尝试更多电子元素', icon: '⚡' },
  { year: '2020', title: '《大娱乐家》', desc: '参加多档综艺节目，展现多面才华', icon: '🎭' },
  { year: '2023', title: '十万伏特巡演', desc: '全国14城巡回演唱会，场场爆满', icon: '🔥' },
  { year: '2025', title: '明日世界巡演', desc: '全球巡回演唱会，从中国走向世界', icon: '🌍' },
]

const activeIndex = ref(-1)
</script>

<template>
  <section id="timeline">
    <h2 class="section-title">音乐历程</h2>
    <div class="timeline-scroll">
      <div class="timeline-track">
        <div class="timeline-line"></div>
        <div
          v-for="(item, index) in timeline"
          :key="item.year"
          class="timeline-item"
          :class="{ active: activeIndex === index }"
          @click="activeIndex = activeIndex === index ? -1 : index"
        >
          <div class="node">
            <span class="icon">{{ item.icon }}</span>
          </div>
          <div class="content">
            <span class="year">{{ item.year }}</span>
            <h3>{{ item.title }}</h3>
            <p v-show="activeIndex === index">{{ item.desc }}</p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
#timeline {
  padding: 80px 40px;
  background: linear-gradient(180deg, #0f0a1a 0%, #0a0a1a 100%);
  overflow: hidden;
}

.section-title {
  font-size: 36px;
  color: var(--pink);
  text-align: center;
  margin-bottom: 60px;
  letter-spacing: 4px;
}

.timeline-scroll {
  overflow-x: auto;
  padding: 20px 0 40px;
  -webkit-overflow-scrolling: touch;
}

.timeline-scroll::-webkit-scrollbar {
  height: 4px;
}

.timeline-scroll::-webkit-scrollbar-thumb {
  background: var(--pink);
  border-radius: 2px;
}

.timeline-track {
  display: flex;
  gap: 20px;
  min-width: max-content;
  padding: 0 40px;
  position: relative;
}

.timeline-line {
  position: absolute;
  top: 45px;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, var(--pink), transparent);
}

.timeline-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 180px;
  cursor: pointer;
  position: relative;
}

.node {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: var(--bg-dark);
  border: 2px solid var(--pink);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  transition: all 0.3s;
  position: relative;
  z-index: 1;
}

.timeline-item:hover .node,
.timeline-item.active .node {
  background: var(--pink);
  box-shadow: 0 0 20px var(--pink-glow);
  transform: scale(1.1);
}

.content {
  text-align: center;
  margin-top: 20px;
}

.year {
  font-size: 14px;
  color: var(--pink);
  letter-spacing: 2px;
}

h3 {
  font-size: 16px;
  margin: 8px 0;
  color: var(--text);
}

p {
  font-size: 13px;
  color: var(--text-muted);
  line-height: 1.6;
  max-width: 180px;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
