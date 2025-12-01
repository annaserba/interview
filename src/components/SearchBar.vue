<template>
  <div class="relative">
    <div class="relative">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Поиск по темам..."
        class="w-full px-4 py-3 pl-12 bg-gray-800 border border-gray-700 rounded-lg text-gray-100 placeholder-gray-500 focus:outline-none focus:border-primary-500 focus:ring-2 focus:ring-primary-500/20 transition-all"
        @input="handleSearch"
      />
      <svg 
        class="absolute left-4 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-500"
        fill="none" 
        stroke="currentColor" 
        viewBox="0 0 24 24"
      >
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
      </svg>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const searchQuery = ref('');

const emit = defineEmits<{
  search: [query: string];
}>();

const handleSearch = () => {
  emit('search', searchQuery.value);
};
</script>
