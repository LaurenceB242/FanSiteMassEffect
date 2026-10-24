<script setup>
import { ref, onMounted } from 'vue'

defineProps({
  title: { type: String, required: true },
  video: { type: String, required: true },
  poster: { type: String, default: '' },
})

const videoEl = ref(null)
const playing = ref(true)

onMounted(() => {
  // Respecte les personnes qui ont désactivé les animations dans leur système
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (reduced) {
    videoEl.value.pause()
    playing.value = false
  }
})

function toggle() {
  if (videoEl.value.paused) {
    videoEl.value.play()
    playing.value = true
  } else {
    videoEl.value.pause()
    playing.value = false
  }
}
</script>

<template>
  <section class="hero">
    <video
      ref="videoEl"
      class="bg"
      :src="video"
      :poster="poster"
      autoplay
      muted
      loop
      playsinline
      aria-hidden="true"
    ></video>

    <h1>{{ title }}</h1>

    <button
      type="button"
      class="toggle"
      :aria-label="playing ? 'Mettre la vidéo en pause' : 'Relancer la vidéo'"
      @click="toggle"
    >
      <svg v-if="playing" viewBox="0 0 24 24" aria-hidden="true">
        <path d="M6 4h4v16H6zM14 4h4v16h-4z" fill="currentColor" />
      </svg>
      <svg v-else viewBox="0 0 24 24" aria-hidden="true">
        <path d="M6 3l15 9-15 9z" fill="currentColor" />
      </svg>
    </button>
  </section>
</template>

<style scoped>
.hero {
  position: relative;
  flex: 1;
  min-height: 0;
  display: grid;
  place-items: center;
  overflow: hidden;
  background: var(--bg-hero);
  text-align: center;
  padding: 1rem;
}
.bg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Voile sombre pour que le titre reste lisible */
.hero::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(rgb(0 0 0 / 0.35), rgb(0 0 0 / 0.65));
}

h1 {
  position: relative;
  z-index: 1;
  margin: 0;
  font: 700 clamp(2.4rem, 9vw, 6rem) var(--font-title);
  text-transform: uppercase;
  color: var(--red);
  letter-spacing: 0.04em;
  text-shadow: 0 2px 24px rgb(0 0 0 / 0.8);
}

.toggle {
  position: absolute;
  z-index: 2;
  right: 1.5rem;
  bottom: 1.5rem;
  width: 3rem;
  height: 3rem;
  padding: 0.8rem;
  border: 1px solid rgb(255 255 255 / 0.5);
  border-radius: 50%;
  background: rgb(0 0 0 / 0.5);
  color: #fff;
  cursor: pointer;
}
.toggle:hover { background: var(--red); border-color: var(--red); }
.toggle svg { width: 100%; height: 100%; }
</style>