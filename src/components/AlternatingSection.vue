<script setup>
defineProps({
  text: { type: String, required: true },
  image: { type: String, default: null },
  alt: { type: String, default: '' },
  reverse: Boolean,
})
</script>

<template>
  <section class="block" :class="{ reverse }">
    <p class="text">{{ text }}</p>
    <img v-if="image" :src="image" :alt="alt" class="media" />
    <div v-else class="media placeholder" role="img" :aria-label="alt">Image</div>
  </section>
</template>

<style scoped>
.block {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 3rem;
  padding: 2rem 0;
}
.text { text-align: center; font-size: 1.5rem; margin: 0; }
.media { width: 100%; max-width: 20rem; aspect-ratio: 3 / 4; object-fit: cover; justify-self: center; }
.reverse .text { order: 2; }

@media (max-width: 700px) {
  .block { grid-template-columns: 1fr; gap: 1.5rem; }
  .reverse .text { order: 0; }
  .text { font-size: 1.25rem; }
}
</style>