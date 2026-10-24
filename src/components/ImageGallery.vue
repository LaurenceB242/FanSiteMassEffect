<script setup>
defineProps({
  items: { type: Array, required: true },
})
</script>

<template>
  <section class="gallery" aria-labelledby="galerie-titre">
    <h2 id="galerie-titre">Galerie</h2>
    <div class="grid">
      <template v-for="(item, i) in items" :key="i">
        <img v-if="item.src" :src="item.src" :alt="item.alt" loading="lazy" />
        <div v-else class="placeholder" role="img" :aria-label="item.alt">{{ item.alt }}</div>
      </template>
    </div>
  </section>
</template>

<style scoped>
.gallery { padding: 3rem 0 5rem; }
h2 { text-align: center; font: 300 2rem var(--font-body); margin: 0 0 2rem; }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(15rem, 1fr)); gap: 1rem; }
.grid > * { aspect-ratio: 4 / 3; width: 100%; object-fit: cover; }
</style>