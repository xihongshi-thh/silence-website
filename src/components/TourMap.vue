<script setup>
import { ref, onMounted } from 'vue'
import { geoMercator, geoPath } from 'd3-geo'

// 带日期的城市数据（标注★为不确定，请核实）
const tours = {
  '十万伏特': {
    status: '已结束',
    color: '#ff6b9d',
    cities: [
      { name: '沈阳', date: '2024.11.30' },
      { name: '太原', date: '2025.01.11' },
      { name: '兰州', date: '2024.12.21' },
      { name: '深圳', date: '2024.12.14' },
      { name: '西安', date: '2025.01.04' },
      { name: '常州', date: '2024.10.19' },
      { name: '南京', date: '2024.10.26' },
      { name: '福州', date: '2024.11.09' },
      { name: '重庆', date: '2024.11.16' },
      { name: '台州', date: '2024.12.07' },
      { name: '厦门', date: '2025.01.18' },
      { name: '杭州', date: '2025.02.15' },
      { name: '南宁', date: '2025.03.01' },
      { name: '广州', date: '2025.03.15' },
    ],
  },
  '明日世界（已开）': {
    status: '已结束',
    color: '#ff8fb1',
    cities: [
      { name: '澳门', date: '2025.04.12' },
      { name: '郑州', date: '2025.05.10' },
      { name: '南昌', date: '2025.05.24' },
      { name: '宜昌', date: '2025.06.07' },
    ],
  },
  '明日世界（即将开）': {
    status: '即将开始',
    color: '#ffeb3b',
    cities: [
      { name: '太原', date: '2025.07.12' },
      { name: '成都', date: '2025.07.26' },
      { name: '上海', date: '2025.08.09' },
      { name: '沈阳', date: '2025.08.23' },
      { name: '南京', date: '2025.09.06' },
      { name: '北京', date: '2025.09.20' },
      { name: '青岛', date: '2025.10.11' },
      { name: '福州', date: '2025.10.25' },
      { name: '杭州', date: '2025.11.08' },
      { name: '深圳', date: '2025.11.22' },
    ],
  },
}

const overseasCities = [
  { name: '纽约', date: '2025.04.26' },
  { name: '西雅图', date: '2025.05.03' },
  { name: '墨尔本', date: '2025.05.17' },
]

const selectedCity = ref(null)
const activeTab = ref('十万伏特')
const provinces = ref([])
const mapReady = ref(false)

const cityCoords = {
  '沈阳': [123.43, 41.80],
  '太原': [112.55, 37.87],
  '兰州': [103.83, 36.06],
  '深圳': [114.07, 22.55],
  '西安': [108.94, 34.26],
  '常州': [119.95, 31.77],
  '南京': [118.78, 32.06],
  '福州': [119.30, 26.08],
  '重庆': [106.55, 29.56],
  '台州': [121.42, 28.66],
  '厦门': [118.09, 24.48],
  '杭州': [120.15, 30.28],
  '南宁': [108.32, 22.82],
  '广州': [113.26, 23.13],
  '澳门': [113.55, 22.20],
  '郑州': [113.65, 34.76],
  '南昌': [115.89, 28.68],
  '宜昌': [111.29, 30.69],
  '成都': [104.07, 30.57],
  '上海': [121.47, 31.23],
  '北京': [116.40, 39.90],
  '青岛': [120.38, 36.07],
}

const width = 800
const height = 700

const projection = geoMercator()
  .center([104, 35])
  .scale(600)
  .translate([width / 2, height / 2])

const pathGen = geoPath().projection(projection)

onMounted(async () => {
  try {
    const res = await fetch('https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json')
    const geojson = await res.json()
    provinces.value = geojson.features
    mapReady.value = true
  } catch (e) {
    console.error('Failed to load map data:', e)
  }
})

function getCityPos(name) {
  const coord = cityCoords[name]
  if (!coord) return null
  return projection(coord)
}

function getCityInfo(name) {
  const result = []
  for (const [tourName, tour] of Object.entries(tours)) {
    const found = tour.cities.find((c) => c.name === name)
    if (found) {
      result.push({ tour: tourName, status: tour.status, color: tour.color, date: found.date })
    }
  }
  return result
}
</script>

<template>
  <section id="tour">
    <h2 class="section-title">巡演地图</h2>

    <div class="tabs">
      <button
        v-for="(tour, name) in tours"
        :key="name"
        :class="{ active: activeTab === name }"
        :style="{ '--tab-color': tour.color }"
        @click="activeTab = name"
      >
        {{ name }}
        <span class="badge" :style="{ background: tour.color }">{{ tour.cities.length }}城</span>
      </button>
    </div>

    <div class="map-container">
      <svg :viewBox="`0 0 ${width} ${height}`" class="china-map">
        <g v-if="mapReady">
          <path
            v-for="(province, i) in provinces"
            :key="i"
            :d="pathGen(province)"
            class="province"
          />
        </g>
        <text v-else :x="width/2" :y="height/2" text-anchor="middle" fill="rgba(255,255,255,0.3)" font-size="16">
          加载地图中...
        </text>

        <!-- City markers -->
        <g v-for="city in tours[activeTab].cities" :key="city.name">
          <template v-if="getCityPos(city.name)">
            <!-- Pulse ring for upcoming -->
            <circle
              v-if="tours[activeTab].status === '即将开始'"
              :cx="getCityPos(city.name)[0]"
              :cy="getCityPos(city.name)[1]"
              fill="none"
              :stroke="tours[activeTab].color"
              stroke-width="1.5"
              opacity="0.6"
            >
              <animate attributeName="r" values="5;15;5" dur="2s" repeatCount="indefinite" />
              <animate attributeName="opacity" values="0.6;0;0.6" dur="2s" repeatCount="indefinite" />
            </circle>
            <!-- City dot -->
            <circle
              :cx="getCityPos(city.name)[0]"
              :cy="getCityPos(city.name)[1]"
              r="5"
              :fill="tours[activeTab].color"
              stroke="white"
              stroke-width="1"
              class="city-dot"
              @click="selectedCity = selectedCity === city.name ? null : city.name"
            />
            <!-- City label + date -->
            <text
              :x="getCityPos(city.name)[0]"
              :y="getCityPos(city.name)[1] - 18"
              text-anchor="middle"
              fill="white"
              font-size="11"
              font-weight="500"
              class="city-label"
            >{{ city.name }}</text>
            <text
              :x="getCityPos(city.name)[0]"
              :y="getCityPos(city.name)[1] - 6"
              text-anchor="middle"
              :fill="tours[activeTab].color"
              font-size="9"
              class="city-label"
            >{{ city.date }}</text>
          </template>
        </g>
      </svg>

      <!-- City info popup -->
      <div v-if="selectedCity" class="city-popup" @click="selectedCity = null">
        <div class="popup-content" @click.stop>
          <h3>{{ selectedCity }}</h3>
          <div v-for="info in getCityInfo(selectedCity)" :key="info.tour" class="tour-info">
            <div class="tour-left">
              <span class="tour-name" :style="{ color: info.color }">{{ info.tour }}</span>
              <span class="tour-date">{{ info.date }}</span>
            </div>
            <span class="tour-status">{{ info.status }}</span>
          </div>
          <button class="close-btn" @click="selectedCity = null">×</button>
        </div>
      </div>
    </div>

    <!-- Overseas cities -->
    <div class="overseas">
      <h3>海外站点</h3>
      <div class="overseas-list">
        <span class="overseas-city" v-for="city in overseasCities" :key="city.name">
          🌏 {{ city.name }} · {{ city.date }}
        </span>
      </div>
    </div>
  </section>
</template>

<style scoped>
#tour {
  padding: 80px 40px;
  background: linear-gradient(180deg, #0a0a1a 0%, #0f0818 100%);
}

.section-title {
  font-size: 36px;
  color: var(--pink);
  text-align: center;
  margin-bottom: 40px;
  letter-spacing: 4px;
}

.tabs {
  display: flex;
  justify-content: center;
  gap: 16px;
  margin-bottom: 40px;
  flex-wrap: wrap;
}

.tabs button {
  padding: 10px 24px;
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.05);
  color: var(--text);
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s;
}

.tabs button.active {
  border-color: var(--tab-color);
  background: rgba(255, 107, 157, 0.1);
  box-shadow: 0 0 15px rgba(255, 107, 157, 0.2);
}

.badge {
  font-size: 11px;
  padding: 2px 8px;
  border-radius: 10px;
  color: white;
}

.map-container {
  max-width: 800px;
  margin: 0 auto;
  position: relative;
}

.china-map {
  width: 100%;
  height: auto;
}

.province {
  fill: none;
  stroke: var(--pink);
  stroke-width: 0.8;
  transition: stroke-width 0.2s;
}

.province:hover {
  stroke-width: 1.5;
  filter: drop-shadow(0 0 3px var(--pink-glow));
}

.city-dot {
  cursor: pointer;
  transition: r 0.2s;
  filter: drop-shadow(0 0 6px var(--pink-glow));
}

.city-dot:hover {
  r: 8;
}

.city-label {
  pointer-events: none;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.9), 0 0 6px rgba(0, 0, 0, 0.6);
}

.city-popup {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.popup-content {
  background: #1a1a2e;
  border: 1px solid var(--pink);
  border-radius: 16px;
  padding: 30px;
  min-width: 280px;
  position: relative;
  animation: popIn 0.2s ease;
}

@keyframes popIn {
  from { transform: scale(0.9); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.popup-content h3 {
  font-size: 24px;
  color: var(--pink);
  margin-bottom: 16px;
}

.tour-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.tour-left {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.tour-name {
  font-size: 14px;
}

.tour-date {
  font-size: 12px;
  color: var(--text-muted);
}

.tour-status {
  font-size: 12px;
  color: var(--text-muted);
  padding: 2px 10px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.close-btn {
  position: absolute;
  top: 12px;
  right: 16px;
  background: none;
  border: none;
  color: var(--text-muted);
  font-size: 24px;
  cursor: pointer;
}

.overseas {
  max-width: 800px;
  margin: 40px auto 0;
  text-align: center;
}

.overseas h3 {
  font-size: 18px;
  color: var(--pink);
  margin-bottom: 16px;
}

.overseas-list {
  display: flex;
  justify-content: center;
  gap: 24px;
  flex-wrap: wrap;
}

.overseas-city {
  font-size: 14px;
  padding: 8px 20px;
  border-radius: 20px;
  background: rgba(255, 107, 157, 0.1);
  border: 1px solid rgba(255, 107, 157, 0.3);
}
</style>
