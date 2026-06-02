<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref(null)
const particles = ref([])
const mouseX = ref(0)
const mouseY = ref(0)
const nameText = ref('')
const fullName = '汪苏泷'
const subtitleText = ref('')
const fullSubtitle = 'Silence Wang'
const fireworks = ref([])
let animId = null

onMounted(() => {
  const cvs = canvas.value
  const ctx = cvs.getContext('2d')
  cvs.width = window.innerWidth
  cvs.height = window.innerHeight

  // Init particles
  for (let i = 0; i < 120; i++) {
    particles.value.push({
      x: Math.random() * cvs.width,
      y: Math.random() * cvs.height,
      size: Math.random() * 2 + 0.5,
      speedX: (Math.random() - 0.5) * 0.5,
      speedY: (Math.random() - 0.5) * 0.5,
      opacity: Math.random() * 0.8 + 0.2,
    })
  }

  function animate() {
    ctx.clearRect(0, 0, cvs.width, cvs.height)

    // Draw particles
    particles.value.forEach((p) => {
      p.x += p.speedX
      p.y += p.speedY

      // Mouse attraction
      const dx = mouseX.value - p.x
      const dy = mouseY.value - p.y
      const dist = Math.sqrt(dx * dx + dy * dy)
      if (dist < 150) {
        p.x += dx * 0.01
        p.y += dy * 0.01
      }

      // Wrap around
      if (p.x < 0) p.x = cvs.width
      if (p.x > cvs.width) p.x = 0
      if (p.y < 0) p.y = cvs.height
      if (p.y > cvs.height) p.y = 0

      ctx.beginPath()
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(255, 107, 157, ${p.opacity})`
      ctx.fill()
    })

    // Draw connections
    particles.value.forEach((a, i) => {
      particles.value.slice(i + 1).forEach((b) => {
        const dist = Math.sqrt((a.x - b.x) ** 2 + (a.y - b.y) ** 2)
        if (dist < 100) {
          ctx.beginPath()
          ctx.moveTo(a.x, a.y)
          ctx.lineTo(b.x, b.y)
          ctx.strokeStyle = `rgba(255, 107, 157, ${0.15 * (1 - dist / 100)})`
          ctx.lineWidth = 0.5
          ctx.stroke()
        }
      })
    })

    // Draw fireworks
    fireworks.value = fireworks.value.filter((f) => f.life > 0)
    fireworks.value.forEach((f) => {
      f.x += f.vx
      f.y += f.vy
      f.vy += 0.05
      f.life--
      ctx.beginPath()
      ctx.arc(f.x, f.y, f.size, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(${f.color}, ${f.life / f.maxLife})`
      ctx.fill()
    })

    animId = requestAnimationFrame(animate)
  }

  animate()

  // Name typewriter effect
  let i = 0
  const nameInterval = setInterval(() => {
    if (i < fullName.length) {
      nameText.value += fullName[i]
      i++
    } else {
      clearInterval(nameInterval)
      // Start subtitle
      let j = 0
      const subInterval = setInterval(() => {
        if (j < fullSubtitle.length) {
          subtitleText.value += fullSubtitle[j]
          j++
        } else {
          clearInterval(subInterval)
        }
      }, 80)
    }
  }, 200)

  window.addEventListener('resize', () => {
    cvs.width = window.innerWidth
    cvs.height = window.innerHeight
  })
})

onUnmounted(() => {
  if (animId) cancelAnimationFrame(animId)
})

function onMouseMove(e) {
  mouseX.value = e.clientX
  mouseY.value = e.clientY
}

function onClick(e) {
  const colors = ['255,107,157', '255,143,177', '255,200,220', '255,255,255']
  for (let i = 0; i < 30; i++) {
    const angle = (Math.PI * 2 * i) / 30
    const speed = 2 + Math.random() * 3
    fireworks.value.push({
      x: e.clientX,
      y: e.clientY,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      size: Math.random() * 3 + 1,
      life: 60 + Math.random() * 30,
      maxLife: 90,
      color: colors[Math.floor(Math.random() * colors.length)],
    })
  }
}
</script>

<template>
  <section id="hero" @mousemove="onMouseMove" @click="onClick">
    <canvas ref="canvas" class="particle-canvas"></canvas>
    <div class="hero-content">
      <h1 class="name">{{ nameText }}<span class="cursor">|</span></h1>
      <p class="subtitle">{{ subtitleText }}</p>
      <div class="scroll-hint">
        <span>向下探索</span>
        <div class="arrow"></div>
      </div>
    </div>
  </section>
</template>

<style scoped>
#hero {
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(ellipse at center, #1a0a1a 0%, #0a0a1a 70%);
  cursor: crosshair;
  overflow: hidden;
}

.particle-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.hero-content {
  position: relative;
  z-index: 1;
  text-align: center;
}

.name {
  font-size: 80px;
  font-weight: 700;
  color: var(--pink);
  text-shadow: 0 0 40px var(--pink-glow), 0 0 80px rgba(255, 107, 157, 0.2);
  letter-spacing: 12px;
  margin-bottom: 16px;
}

.cursor {
  animation: blink 1s infinite;
  font-weight: 300;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

.subtitle {
  font-size: 24px;
  color: var(--text-muted);
  letter-spacing: 8px;
  min-height: 36px;
}

.scroll-hint {
  position: absolute;
  bottom: -120px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: var(--text-muted);
  font-size: 12px;
  letter-spacing: 2px;
}

.arrow {
  width: 20px;
  height: 20px;
  border-right: 2px solid var(--pink);
  border-bottom: 2px solid var(--pink);
  transform: rotate(45deg);
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: rotate(45deg) translateY(0); }
  50% { transform: rotate(45deg) translateY(10px); }
}

@media (max-width: 768px) {
  .name {
    font-size: 48px;
    letter-spacing: 6px;
  }
  .subtitle {
    font-size: 16px;
    letter-spacing: 4px;
  }
}
</style>
