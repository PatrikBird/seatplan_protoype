<script setup lang="ts">
import { ref, computed, watchEffect } from 'vue'
import Spot from './Spot.vue'
import DropZone from './DropZone.vue'
import { type Position, type Participant } from '../types'

const props = defineProps<{
  showChairRow: boolean
}>()

const names = [
  'James', 'John', 'Robert', 'Michael', 'William', 
  'David', 'Richard', 'Joseph', 'Thomas', 'Charles',
  'Christopher', 'Daniel', 'Matthew', 'Anthony', 'Donald', 
  'Mark', 'Tim', 'Patrik', 'Jim', 'Chris', 'Abor', 'Barak', 'Tobi', 'Andi'
]

const participants = ref<Participant[]>(
  names.map((name, index) => ({
    id: index + 1,
    name,
    color: 'blue',
    position: {
      x: index % 8,
      y: 7 - Math.floor(index / 8)
    }
  }))
)

watchEffect(() => {
  console.log('Number of participants:', participants.value.length)
})

const draggedParticipant = ref<Participant | null>(null)
const dragOverPosition = ref<Position | null>(null)
const activeDropZone = ref<{ x: number, y: number } | null>(null)

function isValidDropZone(x: number, y: number): boolean {
  if (!draggedParticipant.value) return false

  // Check if both adjacent spots are occupied
  const leftParticipant = getParticipantAtPosition({ x, y })
  const rightParticipant = getParticipantAtPosition({ x: x + 1, y })

  return leftParticipant !== undefined && rightParticipant !== undefined
}

const room = computed(() => {
  const columns = props.showChairRow ? 9 : 8
  return Array.from({ length: 8 }, (_, y) =>
    Array.from({ length: columns }, (_, x) => ({ x, y }))
  )
})

function getParticipantAtPosition(position: Position): Participant | undefined {
  return participants.value.find(participant => 
    participant.position.x === position.x && participant.position.y === position.y
  )
}

function isValidDropTarget(position: Position): boolean {
  if (!draggedParticipant.value) return false
  
  const targetParticipant = getParticipantAtPosition(position)
  return !targetParticipant || targetParticipant.id !== draggedParticipant.value.id
}

function reorderParticipants(dropX: number, dropY: number) {
  if (!draggedParticipant.value) return

  const participantsInRow = participants.value.filter(p => p.position.y === dropY)
  const sortedParticipants = [...participantsInRow].sort((a, b) => a.position.x - b.position.x)
  
  const currentIndex = sortedParticipants.findIndex(p => p.id === draggedParticipant.value!.id)
  if (currentIndex !== -1) {
    sortedParticipants.splice(currentIndex, 1)
  }

  const insertIndex = sortedParticipants.findIndex(p => p.position.x > dropX)
  const insertPosition = insertIndex === -1 ? sortedParticipants.length : insertIndex

  sortedParticipants.splice(insertPosition, 0, draggedParticipant.value)

  sortedParticipants.forEach((participant, index) => {
    participant.position = {
      x: index,
      y: dropY
    }
  })
}

function handleParticipantDrop(position: Position) {
  if (!draggedParticipant.value) return

  if (activeDropZone.value && isValidDropZone(activeDropZone.value.x, activeDropZone.value.y)) {
    reorderParticipants(activeDropZone.value.x, activeDropZone.value.y)
  } else if (isValidDropTarget(position)) {
    const targetParticipant = getParticipantAtPosition(position)
    const draggedPosition = { ...draggedParticipant.value.position }

    draggedParticipant.value.position = position

    if (targetParticipant && targetParticipant.id !== draggedParticipant.value.id) {
      targetParticipant.position = draggedPosition
    }
  }

  draggedParticipant.value = null
  dragOverPosition.value = null
  activeDropZone.value = null
}

function handleParticipantDragStart(participant: Participant) {
  draggedParticipant.value = participant
}

function handleDragOver(e: DragEvent, position: Position) {
  e.preventDefault()
  if (!isValidDropTarget(position)) {
    dragOverPosition.value = null
    return
  }
  dragOverPosition.value = position
}

function handleDragLeave() {
  dragOverPosition.value = null
}

function isDropTarget(position: Position): boolean {
  return dragOverPosition.value?.x === position.x && 
         dragOverPosition.value?.y === position.y &&
         isValidDropTarget(position)
}

const gridTemplateColumns = computed(() => {
  const totalColumns = props.showChairRow ? 9 : 8
  const columns = []
  for (let i = 0; i < totalColumns; i++) {
    columns.push('1fr')
    if (i < totalColumns - 1) {
      // Add wider gap before the chair column
      if (props.showChairRow && i === totalColumns - 2) {
        columns.push('24px')
      } else {
        columns.push('8px')
      }
    }
  }
  return columns.join(' ')
})
</script>

<template>
  <div class="w-full max-w-4xl mx-auto aspect-square">
    <div 
      class="grid gap-y-4 bg-gray-300 p-6 rounded-lg shadow-xl h-full" 
      :style="{ gridTemplateColumns }"
    >
      <template v-for="row in room" :key="row[0].y">
        <template v-for="(spot, x) in row" :key="`${spot.x}-${spot.y}`">
          <Spot
            :position="spot"
            :participant="getParticipantAtPosition(spot)"
            :is-dark="(spot.x + spot.y) % 2 === 0"
            :is-drop-target="isDropTarget(spot)"
            :is-valid-target="isValidDropTarget(spot)"
            :is-chair="props.showChairRow && x === 8"
            @participant-drop="handleParticipantDrop"
            @participant-drag-start="handleParticipantDragStart"
            @drag-over="handleDragOver"
            @drag-leave="handleDragLeave"
          />
          <DropZone
            v-if="x < (props.showChairRow ? 8 : 7)"
            :is-active="activeDropZone?.x === x && activeDropZone?.y === spot.y"
            :is-dragging="!!draggedParticipant"
            :is-valid="isValidDropZone(x, spot.y)"
            @dragenter="activeDropZone = { x, y: spot.y }"
            @dragleave="activeDropZone = null"
            @drop="handleParticipantDrop(spot)"
          />
        </template>
      </template>
    </div>
  </div>
</template>