<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

interface Heading {
  id: string
  text: string
  level: number
}

const headings = ref<Heading[]>([])
const activeId = ref<string>('')

onMounted(() => {
  // Собрать все заголовки из контента
  const content = document.querySelector('.prose')
  if (!content) return
  
  const headingElements = content.querySelectorAll('h2, h3')
  
  headings.value = Array.from(headingElements).map((heading) => {
    // Создать id если его нет
    if (!heading.id) {
      heading.id = heading.textContent
        ?.toLowerCase()
        .replace(/[^a-zа-я0-9]+/g, '-')
        .replace(/^-|-$/g, '') || ''
    }
    
    return {
      id: heading.id,
      text: heading.textContent || '',
      level: parseInt(heading.tagName[1])
    }
  })
  
  // Отслеживать активный заголовок при скролле
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          activeId.value = entry.target.id
        }
      })
    },
    {
      rootMargin: '-100px 0px -66%',
      threshold: 0
    }
  )
  
  headingElements.forEach((heading) => {
    observer.observe(heading)
  })
  
  // Cleanup
  onUnmounted(() => {
    observer.disconnect()
  })
})

function scrollToHeading(id: string) {
  const element = document.getElementById(id)
  if (element) {
    const offset = 100
    const elementPosition = element.getBoundingClientRect().top
    const offsetPosition = elementPosition + window.pageYOffset - offset
    
    window.scrollTo({
      top: offsetPosition,
      behavior: 'smooth'
    })
  }
}
</script>

<template>
  <nav v-if="headings.length > 0" class="toc">
    <div class="toc-header">
      <h3 class="text-xs font-semibold text-gray-400 uppercase tracking-wider mb-3">
        Содержание
      </h3>
    </div>
    
    <ul class="space-y-1">
      <li
        v-for="heading in headings"
        :key="heading.id"
        :class="[
          'toc-item',
          heading.level === 3 ? 'ml-3' : '',
        ]"
      >
        <a
          :href="`#${heading.id}`"
          @click.prevent="scrollToHeading(heading.id)"
          :class="[
            'block py-1 px-2 rounded transition-all duration-200 line-clamp-2',
            heading.level === 2 ? 'text-sm font-medium' : 'text-xs',
            activeId === heading.id
              ? 'bg-primary-500/20 text-primary-400 border-l-2 border-primary-500 font-semibold'
              : 'text-gray-400 hover:text-gray-300 hover:bg-gray-800/50 border-l-2 border-transparent'
          ]"
        >
          {{ heading.text }}
        </a>
      </li>
    </ul>
  </nav>
</template>

<style scoped>
.toc {
  position: sticky;
  top: 1.5rem;
  max-height: calc(100vh - 3rem);
  overflow-y: auto;
  padding: 1rem;
  background: rgba(17, 24, 39, 0.5);
  border: 1px solid rgba(75, 85, 99, 0.3);
  border-radius: 0.5rem;
  backdrop-filter: blur(10px);
}

.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.toc::-webkit-scrollbar {
  width: 3px;
}

.toc::-webkit-scrollbar-track {
  background: transparent;
}

.toc::-webkit-scrollbar-thumb {
  background: rgba(75, 85, 99, 0.5);
  border-radius: 2px;
}

.toc::-webkit-scrollbar-thumb:hover {
  background: rgba(75, 85, 99, 0.7);
}

@media (max-width: 1280px) {
  .toc {
    display: none;
  }
}
</style>
