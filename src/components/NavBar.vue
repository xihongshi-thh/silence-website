<script setup>
import { ref, onMounted } from 'vue'

const scrolled = ref(false)
const menuOpen = ref(false)

const navItems = [
  { id: 'hero', label: '首页' },
  { id: 'about', label: '关于' },
  { id: 'timeline', label: '音乐历程' },
  { id: 'lyrics', label: '歌词墙' },
  { id: 'albums', label: '专辑' },
  { id: 'tour', label: '巡演地图' },
  { id: 'message', label: '留言墙' },
]

onMounted(() => {
  window.addEventListener('scroll', () => {
    scrolled.value = window.scrollY > 50
  })
})

function scrollTo(id) {
  document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })
  menuOpen.value = false
}
</script>

<template>
  <nav :class="{ scrolled }">
    <div class="nav-inner">
      <div class="logo" @click="scrollTo('hero')">SILENCE</div>
      <button class="menu-toggle" @click="menuOpen = !menuOpen">
        <span :class="{ open: menuOpen }"></span>
      </button>
      <ul :class="{ open: menuOpen }">
        <li v-for="item in navItems" :key="item.id" @click="scrollTo(item.id)">
          {{ item.label }}
        </li>
      </ul>
    </div>
  </nav>
</template>

<style scoped>
nav {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  padding: 20px 40px;
  transition: all 0.3s;
}

nav.scrolled {
  background: rgba(10, 10, 26, 0.95);
  backdrop-filter: blur(10px);
  padding: 12px 40px;
  box-shadow: 0 2px 20px rgba(255, 107, 157, 0.1);
}

.nav-inner {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo {
  font-size: 24px;
  font-weight: 700;
  color: var(--pink);
  cursor: pointer;
  letter-spacing: 4px;
}

ul {
  display: flex;
  list-style: none;
  gap: 32px;
}

li {
  cursor: pointer;
  font-size: 14px;
  color: var(--text-muted);
  transition: color 0.3s;
  letter-spacing: 1px;
}

li:hover {
  color: var(--pink);
}

.menu-toggle {
  display: none;
  background: none;
  border: none;
  cursor: pointer;
  width: 30px;
  height: 24px;
  position: relative;
}

.menu-toggle span,
.menu-toggle span::before,
.menu-toggle span::after {
  display: block;
  width: 100%;
  height: 2px;
  background: var(--pink);
  position: absolute;
  transition: all 0.3s;
}

.menu-toggle span {
  top: 50%;
  transform: translateY(-50%);
}

.menu-toggle span::before {
  content: '';
  top: -8px;
}

.menu-toggle span::after {
  content: '';
  top: 8px;
}

.menu-toggle span.open {
  background: transparent;
}

.menu-toggle span.open::before {
  top: 0;
  transform: rotate(45deg);
}

.menu-toggle span.open::after {
  top: 0;
  transform: rotate(-45deg);
}

@media (max-width: 768px) {
  nav {
    padding: 16px 20px;
  }

  .menu-toggle {
    display: block;
  }

  ul {
    position: fixed;
    top: 0;
    right: -100%;
    width: 70%;
    height: 100vh;
    background: rgba(10, 10, 26, 0.98);
    flex-direction: column;
    padding: 80px 40px;
    gap: 24px;
    transition: right 0.3s;
  }

  ul.open {
    right: 0;
  }
}
</style>
