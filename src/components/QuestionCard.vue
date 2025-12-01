<template>
  <a 
    :href="url"
    class="block group bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-primary-500/50 transition-all duration-300 hover:shadow-lg hover:shadow-primary-500/10 cursor-pointer"
  >
    <div class="flex items-start justify-between mb-3">
      <h3 class="text-xl font-semibold text-gray-100 group-hover:text-primary-400 transition-colors">
        {{ title }}
      </h3>
      <span 
        v-if="difficulty"
        :class="difficultyClass"
        class="px-2 py-1 rounded text-xs font-medium border shrink-0 ml-3"
      >
        {{ difficultyLabel }}
      </span>
    </div>
    
    <p v-if="description" class="text-gray-400 mb-4 line-clamp-2">
      {{ description }}
    </p>
    
    <div class="flex items-center justify-between">
      <div class="flex flex-wrap gap-2">
        <span 
          v-for="tag in tags" 
          :key="tag"
          class="px-2 py-1 bg-gray-900 text-gray-300 rounded text-xs border border-gray-700"
        >
          #{{ tag }}
        </span>
      </div>
      
      <span class="text-primary-400 group-hover:text-primary-300 transition-colors flex items-center">
        <span class="mr-1">Читать</span>
        <svg class="w-4 h-4 group-hover:translate-x-1 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
        </svg>
      </span>
    </div>
  </a>
</template>

<script setup lang="ts">
import { computed } from 'vue';

interface Props {
  title: string;
  description?: string;
  url: string;
  difficulty?: 'easy' | 'medium' | 'hard';
  tags?: string[];
}

const props = withDefaults(defineProps<Props>(), {
  tags: () => [],
});

const difficultyClass = computed(() => {
  const classes = {
    easy: 'bg-green-500/20 text-green-400 border-green-500/30',
    medium: 'bg-yellow-500/20 text-yellow-400 border-yellow-500/30',
    hard: 'bg-red-500/20 text-red-400 border-red-500/30',
  };
  return props.difficulty ? classes[props.difficulty] : '';
});

const difficultyLabel = computed(() => {
  const labels = {
    easy: 'Легко',
    medium: 'Средне',
    hard: 'Сложно',
  };
  return props.difficulty ? labels[props.difficulty] : '';
});
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
