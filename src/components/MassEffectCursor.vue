<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const active = ref(false)    // seulement avec une vraie souris
const visible = ref(false)   // la souris est dans la fenêtre
const hovering = ref(false)  // au-dessus d'un lien ou d'un bouton
const pulses = ref([])

const canvas = ref(null)
const reticle = ref(null)
const dot = ref(null)

const BLUE = '58, 155, 255'
const RED = '255, 42, 54'

let ctx, raf
let w = 0
let h = 0
let pulseId = 0
const mouse = { x: -100, y: -100 }
const ring = { x: -100, y: -100 }
const particles = []

function resize() {
  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  w = window.innerWidth
  h = window.innerHeight
  canvas.value.width = w * dpr
  canvas.value.height = h * dpr
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
}

function onMove(e) {
  if (!visible.value) {
    ring.x = e.clientX
    ring.y = e.clientY
    visible.value = true
  }
  const dist = Math.hypot(e.clientX - mouse.x, e.clientY - mouse.y)
  mouse.x = e.clientX
  mouse.y = e.clientY

  // Plus la souris va vite, plus il y a de particules
  const count = Math.min(4, Math.ceil(dist / 10))
  for (let i = 0; i < count; i++) {
    if (particles.length > 160) particles.shift()
    particles.push({
      x: mouse.x + (Math.random() - 0.5) * 6,
      y: mouse.y + (Math.random() - 0.5) * 6,
      vx: (Math.random() - 0.5) * 0.8,
      vy: (Math.random() - 0.5) * 0.8 - 0.3,
      size: 1 + Math.random() * 2.2,
      life: 1,
      color: hovering.value ? RED : BLUE,
    })
  }
}

function onOver(e) {
  hovering.value = !!e.target.closest?.('a, button, [role="button"]')
}

function onDown() {
  const id = ++pulseId
  pulses.value.push({ id, x: mouse.x, y: mouse.y, red: hovering.value })
  setTimeout(() => {
    pulses.value = pulses.value.filter((p) => p.id !== id)
  }, 700)
}

const onLeave = () => (visible.value = false)
const onEnter = () => (visible.value = true)

function frame() {
  // Le réticule rattrape la souris en douceur (0.18 = vitesse de rattrapage)
  ring.x += (mouse.x - ring.x) * 0.18
  ring.y += (mouse.y - ring.y) * 0.18
  reticle.value.style.transform = `translate3d(${ring.x}px, ${ring.y}px, 0) translate(-50%, -50%)`
  dot.value.style.transform = `translate3d(${mouse.x}px, ${mouse.y}px, 0) translate(-50%, -50%)`

  ctx.clearRect(0, 0, w, h)
  ctx.globalCompositeOperation = 'lighter' // les particules s'additionnent en lumière
  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i]
    p.life -= 0.022
    if (p.life <= 0) {
      particles.splice(i, 1)
      continue
    }
    p.x += p.vx
    p.y += p.vy
    p.vy -= 0.008 // les particules montent doucement

    ctx.fillStyle = `rgba(${p.color}, ${p.life * 0.15})`
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.size * 4, 0, Math.PI * 2)
    ctx.fill()

    ctx.fillStyle = `rgba(${p.color}, ${p.life * 0.9})`
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2)
    ctx.fill()
  }
  raf = requestAnimationFrame(frame)
}

onMounted(() => {
  const mouseOnly = window.matchMedia('(hover: hover) and (pointer: fine)').matches
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (!mouseOnly || reduced) return // pas de curseur perso sur mobile ou si animations réduites

  active.value = true
  document.documentElement.classList.add('has-custom-cursor')
  ctx = canvas.value.getContext('2d')
  resize()

  window.addEventListener('resize', resize)
  window.addEventListener('mousemove', onMove)
  window.addEventListener('mouseover', onOver)
  window.addEventListener('mousedown', onDown)
  document.documentElement.addEventListener('mouseleave', onLeave)
  document.documentElement.addEventListener('mouseenter', onEnter)
  frame()
})

onBeforeUnmount(() => {
  cancelAnimationFrame(raf)
  document.documentElement.classList.remove('has-custom-cursor')
  window.removeEventListener('resize', resize)
  window.removeEventListener('mousemove', onMove)
  window.removeEventListener('mouseover', onOver)
  window.removeEventListener('mousedown', onDown)
  document.documentElement.removeEventListener('mouseleave', onLeave)
  document.documentElement.removeEventListener('mouseenter', onEnter)
})
</script>

<template>
  <div v-show="active" class="me-cursor" :class="{ visible, hovering }" aria-hidden="true">
    <canvas ref="canvas" class="me-trail"></canvas>

    <div ref="reticle" class="me-reticle">
    <div class="me-shape">
        <span class="me-ring"></span>
        <span class="me-ring-dash"></span>
    </div>
    </div>

    <div ref="dot" class="me-dot"></div>

    <span
      v-for="p in pulses"
      :key="p.id"
      class="me-pulse"
      :class="{ red: p.red }"
      :style="{ left: p.x + 'px', top: p.y + 'px' }"
    ></span>
  </div>
</template>

<style>
/* On cache le curseur système, sauf dans la fenêtre vidéo (dialog) */
html.has-custom-cursor,
html.has-custom-cursor * { cursor: none !important; }
html.has-custom-cursor dialog,
html.has-custom-cursor dialog * { cursor: auto !important; }

.me-cursor {
  --c: #3a9bff; /* Paragon */
  position: fixed;
  inset: 0;
  z-index: 9999;
  pointer-events: none;
  color: var(--c);
  opacity: 0;
  transition: opacity 0.25s, color 0.2s;
}
.me-cursor.visible { opacity: 1; }
.me-cursor.hovering { --c: #ff2a36; } /* Renégat */

/* Traînée de particules */
.me-trail {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
}

/* Conteneur du cercle (suit la souris avec un léger retard) */
.me-reticle {
  position: absolute;
  left: 0;
  top: 0;
  width: 2.6rem;
  height: 2.6rem;
}

.me-shape {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.25s cubic-bezier(0.2, 0.8, 0.2, 1);
}
.me-cursor.hovering .me-shape { transform: scale(1); }

/* Le cercle principal */
.me-ring {
  position: absolute;
  inset: 0;
  border: 2px solid currentColor;
  border-radius: 50%;
  box-shadow: 0 0 12px currentColor;
  transition: background 0.2s;
}
.me-cursor.hovering .me-ring { background: rgb(255 42 54 / 0.12); }

/* L'anneau en pointillés qui tourne autour */
.me-ring-dash {
  position: absolute;
  inset: -0.45rem;
  border: 2px dashed currentColor;
  border-radius: 50%;
  opacity: 0.6;
  animation: me-spin 8s linear infinite;
}
.me-cursor.hovering .me-ring-dash { animation-duration: 2.5s; }

/* Le point central (suit la souris sans retard) */
.me-dot {
  position: absolute;
  left: 0;
  top: 0;
  width: 0.4rem;
  height: 0.4rem;
  border-radius: 50%;
  background: currentColor;
  box-shadow: 0 0 10px currentColor;
}

/* L'onde au clic */
.me-pulse {
  position: absolute;
  width: 1rem;
  height: 1rem;
  border: 1px solid #3a9bff;
  border-radius: 50%;
  animation: me-pulse 0.7s ease-out forwards;
}
.me-pulse.red { border-color: #ff2a36; }

@keyframes me-spin {
  to { transform: rotate(360deg); }
}
@keyframes me-pulse {
  from { transform: translate(-50%, -50%) scale(0.3); opacity: 1; }
  to { transform: translate(-50%, -50%) scale(4); opacity: 0; }
}
</style>