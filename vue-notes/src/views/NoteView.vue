<template>
  <div class="min-h-screen p-8 relative z-10">
    <div class="max-w-4xl mx-auto">
      <button 
        @click="router.push('/')"
        class="mb-8 flex items-center text-gray-400 hover:text-white transition-colors group"
      >
        <ArrowLeftIcon class="w-5 h-5 mr-2 group-hover:-translate-x-1 transition-transform" />
        Back to Topics
      </button>

      <div v-if="currentTopic" class="bg-black/40 backdrop-blur-md border border-white/10 rounded-2xl p-8 md:p-12 shadow-2xl">
        <h1 class="text-4xl font-bold mb-8 text-transparent bg-clip-text bg-gradient-to-r from-white to-gray-400">
          {{ currentTopic.title }}
        </h1>
        
        <div v-if="loading" class="flex justify-center py-12">
          <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-white"></div>
        </div>
        
        <div v-else-if="error" class="text-red-400 py-12 text-center">
          {{ error }}
        </div>

        <MarkdownRenderer v-else :content="content" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ArrowLeftIcon } from 'lucide-vue-next'
import { topics } from '../assets/content/index.js'
import MarkdownRenderer from '../components/MarkdownRenderer.vue'

const route = useRoute()
const router = useRouter()

const content = ref('')
const loading = ref(true)
const error = ref(null)

const currentTopic = computed(() => {
  return topics.find(t => t.id === route.params.id)
})

const loadContent = async () => {
  if (!currentTopic.value) {
    error.value = 'Topic not found'
    loading.value = false
    return
  }

  loading.value = true
  error.value = null

  try {
    // In Vite, we can import markdown files as raw strings using ?raw
    // However, dynamic imports with variables are tricky in Vite.
    // We'll use a glob import to get all markdown files.
    const modules = import.meta.glob('../assets/content/*.md', { query: '?raw', import: 'default' })
    
    const path = `../assets/content/${currentTopic.value.file}`
    
    if (modules[path]) {
      content.value = await modules[path]()
    } else {
      throw new Error('File not found')
    }
  } catch (e) {
    console.error(e)
    error.value = 'Failed to load content'
  } finally {
    loading.value = false
  }
}

onMounted(loadContent)
watch(() => route.params.id, loadContent)
</script>
