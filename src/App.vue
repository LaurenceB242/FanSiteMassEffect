<script setup>
import AppHeader from '@/components/AppHeader.vue'
import AppFooter from '@/components/AppFooter.vue'
import MassEffectCursor from '@/components/MassEffectCursor.vue'
</script>

<template>
  <MassEffectCursor />
  <AppHeader />
  <main>
    <RouterView />
  </main>
  <AppFooter />
</template>