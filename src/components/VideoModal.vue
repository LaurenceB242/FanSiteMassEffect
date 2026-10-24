<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  open: Boolean,
  src: { type: String, default: '' },
})
const emit = defineEmits(['close'])

const dialog = ref(null)

watch(
  () => props.open,
  (isOpen) => {
    if (!dialog.value) return
    if (isOpen && !dialog.value.open) dialog.value.showModal()
    if (!isOpen && dialog.value.open) dialog.value.close()
  },
)

function onBackdrop(event) {
  if (event.target === dialog.value) emit('close')
}
</script>

<template>
  <dialog ref="dialog" class="modal" @close="emit('close')" @click="onBackdrop">
    <button type="button" class="close" aria-label="Fermer" @click="emit('close')">×</button>
    <video v-if="open" :src="src" controls autoplay playsinline>
      Votre navigateur ne lit pas cette vidéo.
    </video>
  </dialog>
</template>

<style scoped>
.modal { width: min(90vw, 70rem); padding: 0; border: 0; background: #000; }
.modal::backdrop { background: rgb(0 0 0 / 0.85); }
video { width: 100%; aspect-ratio: 16 / 9; background: #000; }
.close {
  position: absolute;
  top: 0.4rem;
  right: 0.8rem;
  z-index: 1;
  background: none;
  border: 0;
  color: #fff;
  font-size: 2rem;
  cursor: pointer;
}
</style>