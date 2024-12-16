<script setup lang="ts">
import { computed } from 'vue'
import NamePlaceholder from './NamePlaceholder.vue'
import type { Position, Participant } from '../types'

const props = defineProps<{
  position: Position
  participant?: Participant
  isDark: boolean
  isDropTarget: boolean
  isValidTarget: boolean
  isChair?: boolean
}>()

const emit = defineEmits<{
  (e: 'participantDrop', position: Position): void
  (e: 'participantDragStart', participant: Participant): void
  (e: 'dragOver', event: DragEvent, position: Position): void
  (e: 'dragLeave'): void
}>()

const spotClass = computed(() => ({
  'bg-white': !props.isDark && !props.isChair,
  'bg-gray-100': props.isDark && !props.isChair,
  'bg-blue-50': props.isChair,
  'ring-2 ring-indigo-400 ring-opacity-50': props.participant,
  'ring-4 ring-blue-500 ring-opacity-75 z-40': props.isDropTarget && props.isValidTarget,
  'scale-105': props.isDropTarget && props.isValidTarget
}))

const positionLabel = computed(() => {
  if (props.isChair) {
    return `Chair ${8 - props.position.y}`
  }
  const row = String.fromCharCode(65 + (7 - props.position.y))
  const col = props.position.x + 1
  return `${row}${col}`
})

function handleDragOver(e: DragEvent) {
  e.preventDefault()
  emit('dragOver', e, props.position)
}

function handleDragLeave() {
  emit('dragLeave')
}

function handleDrop(e: DragEvent) {
  e.preventDefault()
  emit('participantDrop', props.position)
}
</script>

<template>
  <div
    class="aspect-square flex items-center justify-center transition-all duration-200 rounded-lg overflow-hidden relative group"
    :class="[
      spotClass,
      { 'cursor-not-allowed': !isValidTarget }
    ]"
    @dragover="handleDragOver"
    @dragleave="handleDragLeave"
    @drop="handleDrop"
  >
    <div 
      class="absolute text-gray-600 group-hover:text-gray-800 top-1 left-1 text-xs font-mono z-10 transition-opacity duration-200"
    >
      {{ positionLabel }}
    </div>
    <div class="relative z-0 w-full h-full">
      <NamePlaceholder
        v-if="participant"
        :participant="participant"
        @drag-start="emit('participantDragStart', participant)"
      />
    </div>
  </div>
</template>