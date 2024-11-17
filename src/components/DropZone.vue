<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  isActive: boolean
  isDragging: boolean
  isValid: boolean
}>()

const dropZoneClass = computed(() => ({
  'bg-green-500 shadow-lg scale-150': props.isActive && props.isValid,
  'bg-blue-300 opacity-0': !props.isActive && props.isDragging && props.isValid,
  'bg-transparent': !props.isDragging || !props.isValid,
  'z-50': props.isActive || props.isDragging,
  'cursor-not-allowed': !props.isValid
}))
</script>

<template>
  <div 
    class="w-2 h-full transition-all duration-200 rounded-full transform" 
    :class="dropZoneClass"
    @dragover.prevent
    @drop.prevent="$emit('drop')"
  />
</template>