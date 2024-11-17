<script setup lang="ts">
import type { Participant } from '../types'

const props = defineProps<{
  participant: Participant
}>()

const emit = defineEmits<{
  (e: 'dragStart', participant: Participant): void
}>()

function handleDragStart() {
  emit('dragStart', props.participant)
}
</script>

<template>
  <div
    class="w-full h-full p-3 flex items-center justify-center rounded-md text-black text-sm font-medium cursor-move select-none transition-transform hover:scale-105 bg-blue-200 hover:bg-blue-300"
    draggable="true"
    @dragstart="handleDragStart"
  >
    {{ participant.name }}
  </div>
</template>
