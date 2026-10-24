<script setup>
import AlternatingSection from '@/components/AlternatingSection.vue'
import ImageGallery from '@/components/ImageGallery.vue'
import { sections, gallery } from '@/data/bioware'
</script>

<template>
  <div class="container page">
    <h1>BioWare</h1>
    <AlternatingSection
      v-for="(s, i) in sections"
      :key="i"
      :text="s.text"
      :image="s.image"
      :alt="s.alt"
      :reverse="i % 2 === 1"
    />
    <ImageGallery :items="gallery" />
  </div>
</template>

<style scoped>
.page { padding-top: 3rem; }
h1 { text-align: center; font: 400 2.2rem var(--font-body); margin: 0 0 2rem; }
</style>