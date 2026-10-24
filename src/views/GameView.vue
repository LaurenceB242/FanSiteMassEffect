<script setup>
import { computed, ref } from 'vue'
import { games } from '@/data/games'
import VideoModal from '@/components/VideoModal.vue'

const props = defineProps({ id: String })

const game = computed(() => games.find((g) => g.id === props.id))
const showVideo = ref(false)
</script>

<template>
  <div v-if="game" class="container page">
    <h1>{{ game.title }}</h1>
    <p class="year">{{ game.year }} – BioWare</p>

    <div class="media-wrap">
      <img v-if="game.cover" :src="game.cover" :alt="`Visuel de ${game.title}`" class="media" />
      <div v-else class="media placeholder">Image de {{ game.title }}</div>
      <button type="button" class="play" aria-label="Lire la vidéo" @click="showVideo = true">▶</button>
    </div>

    <p class="tagline">{{ game.tagline }}</p>
    <p v-for="(p, i) in game.text" :key="i" class="explication">{{ p }}</p>

    <VideoModal :open="showVideo" :src="game.video" @close="showVideo = false" />
  </div>
  <p v-else class="container page">Jeu introuvable.</p>
</template>

<style scoped>
.page { padding-top: 3rem; padding-bottom: 5rem; text-align: center; }
h1 { font: 700 clamp(1.8rem, 5vw, 3rem) var(--font-title); color: var(--red); margin: 0; text-transform: uppercase; }
.year { opacity: 0.65; margin: 0.3rem 0 2rem; }
.media-wrap { position: relative; max-width: 48rem; margin: 0 auto 2rem; }
.media { width: 100%; aspect-ratio: 16 / 9; object-fit: cover; }
.play {
  position: absolute;
  inset: 50% auto auto 50%;
  transform: translate(-50%, -50%);
  width: 4.5rem;
  height: 4.5rem;
  border-radius: 50%;
  border: 2px solid #fff;
  background: rgb(0 0 0 / 0.55);
  color: #fff;
  font-size: 1.6rem;
  cursor: pointer;
}
.play:hover { background: var(--red); border-color: var(--red); }
.tagline { font-size: 1.5rem; }
.explication { max-width: 40rem; margin: 1rem auto; }
</style>