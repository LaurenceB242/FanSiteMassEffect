<script setup>
import { ref } from 'vue'

defineProps({
  text: { type: String, required: true },
  image: { type: String, default: null },
  alt: { type: String, default: '' },
  reverse: Boolean,
})

const frame = ref(null)

// Intensité de l'effet : à ajuster
const TILT = 8      // inclinaison maximale du cadre, en degrés
const SHIFT = 18    // décalage maximal de l'image, en pixels

function onMove(e) {
  const el = frame.value
  const rect = el.getBoundingClientRect()
  // x et y vont de -0.5 (bord gauche/haut) à +0.5 (bord droit/bas)
  const x = (e.clientX - rect.left) / rect.width - 0.5
  const y = (e.clientY - rect.top) / rect.height - 0.5

  el.style.setProperty('--rx', `${-y * TILT}deg`)
  el.style.setProperty('--ry', `${x * TILT}deg`)
  el.style.setProperty('--tx', `${-x * SHIFT}px`)
  el.style.setProperty('--ty', `${-y * SHIFT}px`)
}

function onLeave() {
  const el = frame.value
  el.style.setProperty('--rx', '0deg')
  el.style.setProperty('--ry', '0deg')
  el.style.setProperty('--tx', '0px')
  el.style.setProperty('--ty', '0px')
}
</script>

<template>
  <section class="block" :class="{ reverse }">
    <p class="text">{{ text }}</p>

    <div ref="frame" class="frame" @mousemove="onMove" @mouseleave="onLeave">
      <img v-if="image" :src="image" :alt="alt" class="media" />
      <div v-else class="media placeholder" role="img" :aria-label="alt">Image</div>
    </div>
  </section>
</template>

<style scoped>
.block {
  display: grid;
  grid-template-columns: 1fr 1.4fr;
  align-items: center;
  gap: 3rem;
  padding: 2rem 0;
}
.block.reverse { grid-template-columns: 1.4fr 1fr; }

.text { text-align: center; font-size: 1.5rem; margin: 0; }
.reverse .text { order: 2; }

/* Le cadre : coins arrondis + inclinaison 3D */
.frame {
  border-radius: 1.2rem;
  overflow: hidden;              /* l'image zoomée reste dans les coins arrondis */
  box-shadow: 0 10px 30px rgb(0 0 0 / 0.5);
  transform: perspective(900px) rotateX(var(--rx, 0deg)) rotateY(var(--ry, 0deg));
  transition: transform 0.5s cubic-bezier(0.2, 0.8, 0.2, 1), box-shadow 0.3s;
  will-change: transform;
}

/* L'image : toute visible à son format d'origine */
.media {
  display: block;
  width: 100%;
  height: auto;
  transform: scale(1) translate(0, 0);
  transition: transform 0.5s cubic-bezier(0.2, 0.8, 0.2, 1);
}
img.media { object-fit: contain; }
div.media { aspect-ratio: 16 / 9; }

/* L'effet n'existe qu'avec une vraie souris */
@media (hover: hover) and (pointer: fine) {
  .frame:hover {
    transition: transform 0.12s ease-out, box-shadow 0.3s; /* réactif pendant le mouvement */
    box-shadow: 0 18px 45px #c4101c;        /* halo bleu */
  }
  .frame:hover .media {
    transform: scale(1.1) translate(var(--tx, 0px), var(--ty, 0px));
    transition: transform 0.12s ease-out;
  }
}

@media (prefers-reduced-motion: reduce) {
  .frame,
  .frame:hover { transform: none; transition: none; }
  .frame:hover .media { transform: none; }
}

@media (max-width: 700px) {
  .block,
  .block.reverse { grid-template-columns: 1fr; gap: 1.5rem; }
  .reverse .text { order: 0; }
  .text { font-size: 1.25rem; }
}
</style>