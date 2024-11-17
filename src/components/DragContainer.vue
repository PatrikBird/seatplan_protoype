<script setup lang="ts">
import { ref } from 'vue'

interface Item {
  id: number
  text: string
  color: string
}

interface Container {
  id: number
  title: string
  items: Item[]
}

const props = defineProps<{
  container: Container
  containers: Container[]
}>()

const draggedItem = ref<Item | null>(null)
const isDraggingOver = ref(false)

function handleDragStart(item: Item) {
  draggedItem.value = item
}

function handleDragOver(e: DragEvent) {
  e.preventDefault()
  isDraggingOver.value = true
}

function handleDragLeave() {
  isDraggingOver.value = false
}

function handleDrop(e: DragEvent, targetContainer: Container) {
  e.preventDefault()
  isDraggingOver.value = false
  
  if (!draggedItem.value) return

  // Find source container
  const sourceContainer = props.containers.find(c => 
    c.items.some(item => item.id === draggedItem.value?.id)
  )

  if (!sourceContainer) return

  // Remove from source
  sourceContainer.items = sourceContainer.items.filter(
    item => item.id !== draggedItem.value?.id
  )

  // Add to target
  targetContainer.items.push(draggedItem.value)
  draggedItem.value = null
}
</script>

<template>
  <div
    class="bg-white rounded-lg shadow-lg p-4 min-w-[300px]"
    @dragover="handleDragOver"
    @dragleave="handleDragLeave"
    @drop="handleDrop($event, container)"
    :class="{ 'ring-2 ring-blue-400 ring-opacity-50': isDraggingOver }"
  >
    <h2 class="text-xl font-semibold mb-4 text-gray-700">{{ container.title }}</h2>
    <div class="space-y-2">
      <div
        v-for="item in container.items"
        :key="item.id"
        class="p-3 rounded-md shadow-sm cursor-move transition-transform duration-200 hover:scale-102"
        :class="[item.color]"
        draggable="true"
        @dragstart="handleDragStart(item)"
      >
        {{ item.text }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.hover\:scale-102:hover {
  transform: scale(1.02);
}
</style>