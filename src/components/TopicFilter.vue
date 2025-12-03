<script setup lang="ts">
import { ref, computed } from 'vue'

interface Props {
  categories: string[]
  difficulties: string[]
}

const props = defineProps<Props>()

const emit = defineEmits<{
  filter: [filters: { category: string | null; difficulty: string | null }]
}>()

const selectedCategory = ref<string | null>(null)
const selectedDifficulty = ref<string | null>(null)

const difficultyColors = {
  easy: 'bg-green-500/20 text-green-400 border-green-500/30 hover:bg-green-500/30',
  medium: 'bg-yellow-500/20 text-yellow-400 border-yellow-500/30 hover:bg-yellow-500/30',
  hard: 'bg-red-500/20 text-red-400 border-red-500/30 hover:bg-red-500/30'
}

function selectCategory(category: string | null) {
  selectedCategory.value = category
  emitFilter()
}

function selectDifficulty(difficulty: string | null) {
  selectedDifficulty.value = difficulty
  emitFilter()
}

function emitFilter() {
  emit('filter', {
    category: selectedCategory.value,
    difficulty: selectedDifficulty.value
  })
  
  // Отправить custom event для Astro
  if (typeof window !== 'undefined') {
    window.dispatchEvent(new CustomEvent('topic-filter', {
      detail: {
        category: selectedCategory.value,
        difficulty: selectedDifficulty.value
      }
    }))
  }
}

function resetFilters() {
  selectedCategory.value = null
  selectedDifficulty.value = null
  emitFilter()
}

const hasActiveFilters = computed(() => {
  return selectedCategory.value !== null || selectedDifficulty.value !== null
})
</script>

<template>
  <div class="mb-8 space-y-6">
    <!-- Категории -->
    <div>
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-sm font-semibold text-gray-300 uppercase tracking-wider">
          Категории
        </h3>
        <button
          v-if="hasActiveFilters"
          @click="resetFilters"
          class="text-sm text-primary-400 hover:text-primary-300 transition-colors"
        >
          Сбросить фильтры
        </button>
      </div>
      <div class="flex flex-wrap gap-2">
        <button
          @click="selectCategory(null)"
          :class="[
            'px-4 py-2 rounded-lg text-sm font-medium transition-all duration-200 border',
            selectedCategory === null
              ? 'bg-primary-500/30 text-primary-300 border-primary-500/50 shadow-lg shadow-primary-500/20'
              : 'bg-gray-800/50 text-gray-400 border-gray-700/50 hover:bg-gray-800 hover:border-gray-600'
          ]"
        >
          Все
        </button>
        <button
          v-for="category in categories"
          :key="category"
          @click="selectCategory(category)"
          :class="[
            'px-4 py-2 rounded-lg text-sm font-medium transition-all duration-200 border',
            selectedCategory === category
              ? 'bg-primary-500/30 text-primary-300 border-primary-500/50 shadow-lg shadow-primary-500/20'
              : 'bg-gray-800/50 text-gray-400 border-gray-700/50 hover:bg-gray-800 hover:border-gray-600'
          ]"
        >
          {{ category }}
        </button>
      </div>
    </div>

    <!-- Сложность -->
    <div>
      <h3 class="text-sm font-semibold text-gray-300 uppercase tracking-wider mb-3">
        Сложность
      </h3>
      <div class="flex flex-wrap gap-2">
        <button
          @click="selectDifficulty(null)"
          :class="[
            'px-4 py-2 rounded-lg text-sm font-medium transition-all duration-200 border',
            selectedDifficulty === null
              ? 'bg-primary-500/30 text-primary-300 border-primary-500/50 shadow-lg shadow-primary-500/20'
              : 'bg-gray-800/50 text-gray-400 border-gray-700/50 hover:bg-gray-800 hover:border-gray-600'
          ]"
        >
          Все
        </button>
        <button
          v-for="difficulty in difficulties"
          :key="difficulty"
          @click="selectDifficulty(difficulty)"
          :class="[
            'px-4 py-2 rounded-lg text-sm font-medium transition-all duration-200 border capitalize',
            selectedDifficulty === difficulty
              ? difficultyColors[difficulty as keyof typeof difficultyColors]
              : 'bg-gray-800/50 text-gray-400 border-gray-700/50 hover:bg-gray-800 hover:border-gray-600'
          ]"
        >
          {{ difficulty === 'easy' ? 'Легко' : difficulty === 'medium' ? 'Средне' : 'Сложно' }}
        </button>
      </div>
    </div>

    <!-- Активные фильтры -->
    <div v-if="hasActiveFilters" class="flex items-center gap-2 text-sm text-gray-400">
      <span>Фильтры:</span>
      <span v-if="selectedCategory" class="px-3 py-1 bg-primary-500/20 text-primary-400 rounded-full border border-primary-500/30">
        {{ selectedCategory }}
      </span>
      <span v-if="selectedDifficulty" class="px-3 py-1 bg-gray-700/50 text-gray-300 rounded-full border border-gray-600/50 capitalize">
        {{ selectedDifficulty === 'easy' ? 'Легко' : selectedDifficulty === 'medium' ? 'Средне' : 'Сложно' }}
      </span>
    </div>
  </div>
</template>
