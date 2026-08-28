<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'

// ----- danmaku system -----
interface Danmaku {
  id: number
  text: string
  top: number
  duration: number
  color: string
}

const danmakuList = ref<Danmaku[]>([])
let danmakuId = 0

// get random color
function getRandomColor(): string {
  const hue = Math.floor(Math.random() * 360)
  return `hsl(${hue}, 80%, 65%)`
}

// spawn danmaku
function spawnDanmaku() {
  const texts = [
    'TGWGroup~ (◕ヮ◕)ノvv ！',
  ] as const
  const text = texts[Math.floor(Math.random() * texts.length)]!
  const top = Math.random() * (window.innerHeight - 60)
  const duration = 6 + Math.random() * 4 // 6~10秒
  const color = getRandomColor()

  danmakuList.value.push({
    id: danmakuId++,
    text,
    top,
    duration,
    color,
  })

  setTimeout(() => {
    danmakuList.value = danmakuList.value.filter((d) => d.id !== danmakuId - 1)
  }, duration * 1000 + 100)
}

// 持续生成弹幕
let danmakuInterval: number | null = null
onMounted(() => {
  // 立即生成几条
  for (let i = 0; i < 5; i++) {
    setTimeout(spawnDanmaku, i * 800)
  }
  danmakuInterval = window.setInterval(spawnDanmaku, 2500)
})

// ----- popup text -----
interface Popup {
  id: number
  text: string
  x: number
  y: number
  color: string
}

const popupList = ref<Popup[]>([])
let popupId = 0

function playClickSound() {
  const clickSound  = new Audio('/sounds/TGWGroup.mp3')
  clickSound.play().catch(() => {
    // 静默处理错误
  })
}

function handleClick(event: MouseEvent) {
  const x = event.clientX
  const y = event.clientY
  const texts = [
    'TGWGroup~ (◕ヮ◕)ノvv',
  ] as const
  const text = texts[Math.floor(Math.random() * texts.length)]!
  const color = getRandomColor()

  const popup: Popup = {
    id: popupId++,
    text,
    x,
    y,
    color,
  }
  popupList.value.push(popup)

  // 1.5 second clear
  setTimeout(() => {
    popupList.value = popupList.value.filter((p) => p.id !== popup.id)
  }, 1500)

  playClickSound()
}
</script>

<template>
  <div id = "app" @click="handleClick">
    <div class="main-title">
      <div class="title">豊川グループ</div>
    </div>

    <div class="danmaku-container">
      <div
        v-for="item in danmakuList"
        :key="item.id"
        class="danmaku"
        :style="{
          top: item.top + 'px',
          animationDuration: item.duration + 's',
          color: item.color,
        }"
      >
        {{ item.text }}
      </div>
    </div>

    <div class="popup-container">
      <div
        v-for="item in popupList"
        :key="item.id"
        class="popup-text"
        :style="{
          left: item.x + 'px',
          top: item.y + 'px',
          color: item.color,
        }"
      >
        {{ item.text }}
      </div>
    </div>
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  user-select: none;
}

#app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: linear-gradient(135deg, #fce4ec, #f3e5f5, #e8eaf6);
  font-family: 'Comic Sans MS', 'Chalkboard SE', cursive, sans-serif;
  position: relative;
  cursor: pointer;
}

/* ----- 中央主标题 ----- */
.main-title {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  z-index: 10;
  pointer-events: none; /* 允许点击穿透到背景触发点击事件 */
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 3.2rem;
  font-weight: 900;
  color: #1a237e;
  background: linear-gradient(45deg, #ffd54f, #ff8a65, #ce93d8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: glow 1.8s infinite alternate;
}

@keyframes bounce {
  0% { transform: translateY(-6px); }
  100% { transform: translateY(6px); }
}

@keyframes glow {
  0% { filter: drop-shadow(0 0 5px rgba(255, 213, 79, 0.5)); }
  100% { filter: drop-shadow(0 0 25px rgba(206, 147, 216, 0.9)); }
}

/* ----- 弹幕容器 ----- */
.danmaku-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
  overflow: hidden;
}

.danmaku {
  position: absolute;
  white-space: nowrap;
  font-size: 2.0rem;
  font-weight: bold;
  opacity: 0.9;
  animation: scrollRightToLeft linear forwards;
  text-shadow: 0 0 10px rgba(255, 255, 255, 0.6);
}

@keyframes scrollRightToLeft {
  0% {
    transform: translateX(100vw);
    opacity: 0;
  }
  5% {
    opacity: 0.9;
  }
  90% {
    opacity: 0.9;
  }
  100% {
    transform: translateX(-200%);
    opacity: 0;
  }
}

/* ----- 点击弹出文字 ----- */
.popup-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 20;
}

.popup-text {
  position: absolute;
  font-size: 1.2rem;
  font-weight: bold;
  white-space: nowrap;
  transform: translate(-50%, -50%);
  animation: popUpFade 1.5s ease-out forwards;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.8);
}

@keyframes popUpFade {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(0.5);
  }
  30% {
    opacity: 1;
    transform: translate(-50%, -70%) scale(1.2);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -150%) scale(1);
  }
}

/* 手机适配 */
@media (max-width: 600px) {
  .name { font-size: 2.5rem; }
  .action { font-size: 1.8rem; }
  .group { font-size: 3rem; }
  .danmaku { font-size: 1.2rem; }
  .popup-text { font-size: 1.6rem; }
}
</style>
