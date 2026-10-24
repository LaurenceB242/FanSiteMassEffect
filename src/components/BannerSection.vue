<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  title: { type: String, required: true },
  text: { type: String, required: true },
  image: { type: String, default: null },
  alt: { type: String, default: '' },
  layers: { type: Array, default: () => [] }, // [{ src, alt, caption }]
})

const banner = ref(null)
const stage = ref(null)
const layerEls = []
let ticking = false

const clamp = (v) => Math.min(1, Math.max(0, v))

function update() {
  ticking = false
  if (!banner.value || !stage.value) return

  // p : avancement du défilement dans la bannière (0 → 1)
  const rect = banner.value.getBoundingClientRect()
  const stickyTop = parseFloat(getComputedStyle(stage.value).top) || 0
  const distance = rect.height - stage.value.offsetHeight
  const p = clamp(distance > 0 ? (stickyTop - rect.top) / distance : 0)
  banner.value.style.setProperty('--p', p.toFixed(3))

  // Les images démarrent quand le texte est presque sorti (p > 0.2)
  // et sont toutes là à p = 0.9
  const n = props.layers.length
  const q = clamp((p - 0.2) / 0.7) * n
  layerEls.forEach((el, i) => {
    if (el) el.style.setProperty('--r', clamp(q - i).toFixed(3))
  })
}

function onScroll() {
  if (!ticking) {
    ticking = true
    requestAnimationFrame(update)
  }
}

onMounted(() => {
  update()
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('resize', onScroll)
})
onBeforeUnmount(() => {
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('resize', onScroll)
})
</script>

<template>
  <section ref="banner" class="banner" :style="{ '--steps': layers.length }">
    <!-- Scène figée sous le header -->
    <div ref="stage" class="stage">
      <img v-if="image" :src="image" :alt="alt" class="bg" />
      <div class="shade"></div>

      <!-- Images qui se dévoilent au défilement -->
      <figure
        v-for="(l, i) in layers"
        :key="i"
        :ref="(el) => (layerEls[i] = el)"
        class="layer"
      >
        <img v-if="l.src" :src="l.src" :alt="l.alt" />
        <div v-else class="ph" :style="{ '--h': 210 + i * 55 }">{{ l.alt }}</div>
        <figcaption v-if="l.caption" class="cap">{{ l.caption }}</figcaption>
      </figure>
    </div>

    <!-- Texte qui remonte, puis zone de défilement -->
    <div class="content">
      <h1 class="sr-only">{{ title }}</h1>
      <p>{{ text }}</p>
    </div>
  </section>
</template>

<style scoped>
.banner {
  --stage-h: calc(100vh - var(--header-h, 4.5rem));
  --stage-h: calc(100svh - var(--header-h, 4.5rem));
  --start: 62svh;
  --p: 0;
  position: relative;
}

.stage {
  position: sticky;
  top: var(--header-h, 4.5rem);
  height: var(--stage-h);
  overflow: hidden;
  background: var(--bg-hero, #0a0a0a);
}

.bg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
}

.shade {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(ellipse at 50% 110%, rgb(196 16 28 / 0.6), transparent 65%),
    linear-gradient(to top, rgb(14 2 6 / 0.85), transparent 60%),
    rgb(28 2 6 / calc(var(--p) * 0.7));
  box-shadow: inset 0 0 12rem rgb(196 16 28 / 0.5);
}

/* Chaque image se dévoile de bas en haut, avec un léger zoom arrière */
.layer {
  --r: 0;
  position: absolute;
  inset: 0;
  margin: 0;
  overflow: hidden;
  clip-path: inset(calc((1 - var(--r)) * 100%) 0 0 0);
  opacity: calc(var(--r) * 3);
  will-change: clip-path, opacity;
}
.layer img,
.ph {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transform: scale(calc(1.18 - var(--r) * 0.18));
}
.ph {
  display: grid;
  place-items: center;
  font: 700 clamp(1.6rem, 5vw, 3.5rem) var(--font-title, sans-serif);
  text-transform: uppercase;
  color: rgb(255 255 255 / 0.85);
  background: linear-gradient(160deg, hsl(var(--h) 60% 24%), hsl(var(--h) 55% 8%));
}

/* Dégradé sous la légende + légende qui apparaît en fin de dévoilement */
.layer::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgb(6 7 15 / 0.85), transparent 45%);
  pointer-events: none;
}
.cap {
  position: absolute;
  z-index: 1;
  left: 0;
  right: 0;
  bottom: 2.5rem;
  padding: 0 1.5rem;
  text-align: center;
  font-size: clamp(1.1rem, 2.4vw, 1.7rem);
  text-shadow: 0 2px 14px rgb(0 0 0 / 0.9);
  opacity: calc((var(--r) - 0.6) * 2.5);
  transform: translateY(calc((1 - var(--r)) * 1.5rem));
}

/* Le texte remonte, puis la zone vide donne la durée du défilement */
.content {
  position: relative;
  z-index: 1;
  margin-top: calc(var(--start) - var(--stage-h));
  /* 70svh de défilement par image */
  min-height: calc(var(--stage-h) + 25svh + var(--steps, 0) * 70svh);
  padding: 0 1.5rem;
  pointer-events: none;
}
.content p {
  max-width: 44rem;
  margin: 0 auto;
  text-align: center;
  font-size: clamp(1.15rem, 2.4vw, 1.7rem);
  text-shadow:
    0 2px 14px rgb(0 0 0 / 0.9),
    0 0 28px rgb(196 16 28 / 0.6);
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}

/* Mobile : pas d'effet, tout s'empile simplement, sans espace vide */
@media (max-width: 700px) {
  .stage {
    position: relative;
    top: auto;
    height: auto;
    overflow: visible;
  }
  .bg {
    position: static;
    display: block;
    height: auto;
    object-fit: contain;
  }
  .shade { display: none; }
  .layer {
    position: relative;
    clip-path: none;
    opacity: 1;
    aspect-ratio: 16 / 9;
  }
  .layer img,
  .ph { transform: none; }
  .cap { opacity: 1; transform: none; bottom: 1rem; font-size: 1rem; }
  .content {
    margin-top: 0;
    min-height: 0;
    padding: 1.5rem 1.2rem 2rem;
    pointer-events: auto;
  }
}
</style>