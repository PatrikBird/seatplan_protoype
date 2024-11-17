<script setup lang="ts">
import Room from './components/Room.vue'
import { ref } from 'vue'

const showChairRow = ref(false)
</script>

<template>
  <div class="min-h-screen bg-gray-100 p-8">
    <div class="max-w-7xl mx-auto">
      <div class="flex justify-between items-center mb-8">
        <h1 class="text-3xl font-bold text-gray-800">Participant Room</h1>
        <label class="inline-flex items-center cursor-pointer">
          <span class="mr-3 text-sm font-medium text-gray-700">Show Chair Row</span>
          <div class="relative">
            <input 
              type="checkbox" 
              v-model="showChairRow" 
              class="sr-only peer"
            >
            <div class="w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 rounded-full peer peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>
          </div>
        </label>
      </div>
      <Room :show-chair-row="showChairRow" />
    </div>
  </div>
</template>