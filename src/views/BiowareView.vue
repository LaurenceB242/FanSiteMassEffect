<script setup>
import BannerSection from '@/components/BannerSection.vue'
import AlternatingSection from '@/components/AlternatingSection.vue'
import ImageGallery from '@/components/ImageGallery.vue'
import { sections, gallery, reveals } from '@/data/bioware'

// La première section devient la bannière, les autres restent en texte + image
const [banner, ...others] = sections
</script>

<template>
  <BannerSection
    v-if="banner"
    title="BioWare"
    :text="banner.text"
    :image="banner.image"
    :alt="banner.alt"
    :layers="reveals"
  />

  <div class="container page">
    <AlternatingSection
      v-for="(s, i) in others"
      :key="i"
      :text="s.text"
      :image="s.image"
      :alt="s.alt"
      :reverse="i % 2 === 0"
    />
    <ImageGallery :items="gallery" />
  </div>
</template>

<style scoped>
.page { width: 100%; padding-top: 3rem; }
</style>