<template>
  <div class="min-h-screen flex flex-col items-center p-8 pt-16 pb-24 relative z-10">
    <div class="mb-8 text-center fade-in">
      <ShinyText 
        text="Vue Learning Notes" 
        :disabled="false" 
        :speed="3" 
        class="text-5xl md:text-7xl font-bold mb-4" 
      />
      <p class="text-gray-400 text-xl max-w-2xl mx-auto mt-4">
        Explore {{ topics.length }} essential Vue 3 topics
      </p>
      <p class="text-gray-500 text-sm mt-2">
        Click any card to dive into the topic
      </p>
    </div>

    <div class="w-full max-w-7xl mx-auto fade-in-delay px-4">
      <Masonry 
        :items="masonryItems"
        :columns="3"
        :gap="48"
        @item-click="handleItemClick"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { topics } from '../assets/content/index.js'
import ShinyText from '../components/TextAnimations/ShinyText/ShinyText.vue'
import Masonry from '../components/Components/Masonry/Masonry.vue'

const router = useRouter()

const topicColors = [
  { bg1: '#1e3a8a', bg2: '#7c3aed', accent: '#60a5fa' },
  { bg1: '#6b21a8', bg2: '#ec4899', accent: '#c084fc' },
  { bg1: '#be185d', bg2: '#f97316', accent: '#fb923c' },
  { bg1: '#ea580c', bg2: '#facc15', accent: '#fbbf24' },
  { bg1: '#047857', bg2: '#06b6d4', accent: '#34d399' },
  { bg1: '#0891b2', bg2: '#3b82f6', accent: '#22d3ee' },
  { bg1: '#4f46e5', bg2: '#7c3aed', accent: '#818cf8' },
  { bg1: '#d97706', bg2: '#dc2626', accent: '#f59e0b' },
  { bg1: '#0d9488', bg2: '#059669', accent: '#14b8a6' },
  { bg1: '#9333ea', bg2: '#db2777', accent: '#a855f7' }
]

const generateRichCard = (topic, colors, index) => {
  const canvas = document.createElement('canvas')
  canvas.width = 800
  canvas.height = 600
  const ctx = canvas.getContext('2d')
  
  // Background gradient
  const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height)
  gradient.addColorStop(0, colors.bg1)
  gradient.addColorStop(1, colors.bg2)
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, canvas.width, canvas.height)
  
  // Decorative circles
  ctx.globalAlpha = 0.15
  ctx.fillStyle = colors.accent
  ctx.beginPath()
  ctx.arc(650, 100, 200, 0, Math.PI * 2)
  ctx.fill()
  ctx.beginPath()
  ctx.arc(150, 500, 150, 0, Math.PI * 2)
  ctx.fill()
  
  // Add pattern overlay
  ctx.globalAlpha = 0.05
  ctx.fillStyle = 'white'
  for (let i = 0; i < 100; i++) {
    ctx.beginPath()
    ctx.arc(
      Math.random() * canvas.width,
      Math.random() * canvas.height,
      Math.random() * 4 + 1,
      0,
      Math.PI * 2
    )
    ctx.fill()
  }
  
  ctx.globalAlpha = 1.0
  
  // Card border/frame
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)'
  ctx.lineWidth = 3
  ctx.strokeRect(30, 30, canvas.width - 60, canvas.height - 60)
  
  // Topic number badge
  ctx.fillStyle = 'rgba(255, 255, 255, 0.2)'
  ctx.fillRect(50, 50, 80, 60)
  ctx.fillStyle = 'white'
  ctx.font = 'bold 32px Inter, sans-serif'
  ctx.textAlign = 'center'
  ctx.fillText(`${index + 1}`, 90, 90)
  
  // Icon (book symbol)
  ctx.font = '60px Arial'
  ctx.fillText('📚', 400, 150)
  
  // Title with multiline wrapping (narrower width)
  ctx.fillStyle = 'white'
  ctx.font = 'bold 36px Inter, sans-serif'
  ctx.textAlign = 'center'
  ctx.shadowColor = 'rgba(0, 0, 0, 0.3)'
  ctx.shadowBlur = 10
  ctx.shadowOffsetY = 2
  
  const titleWords = topic.title.split(' ')
  let titleLine = ''
  let titleY = 240
  const titleMaxWidth = 500
  
  for (const word of titleWords) {
    const testLine = titleLine + word + ' '
    const metrics = ctx.measureText(testLine)
    
    if (metrics.width > titleMaxWidth && titleLine !== '') {
      ctx.fillText(titleLine.trim(), 400, titleY)
      titleLine = word + ' '
      titleY += 45
    } else {
      titleLine = testLine
    }
  }
  ctx.fillText(titleLine.trim(), 400, titleY)
  
  // Description with multiline wrapping (narrower width)
  ctx.font = '20px Inter, sans-serif'
  ctx.fillStyle = 'rgba(255, 255, 255, 0.9)'
  ctx.shadowBlur = 5
  
  const descWords = topic.description.split(' ')
  let descLine = ''
  let descY = titleY + 55
  const descMaxWidth = 550
  let lineCount = 0
  const maxLines = 5
  
  for (const word of descWords) {
    const testLine = descLine + word + ' '
    const metrics = ctx.measureText(testLine)
    
    if (metrics.width > descMaxWidth && descLine !== '') {
      if (lineCount >= maxLines - 1) {
        ctx.fillText(descLine.trim() + '...', 400, descY)
        break
      }
      ctx.fillText(descLine.trim(), 400, descY)
      descLine = word + ' '
      descY += 32
      lineCount++
    } else {
      descLine = testLine
    }
  }
  
  if (lineCount < maxLines) {
    ctx.fillText(descLine.trim(), 400, descY)
  }
  
  // "Click to Learn" indicator
  ctx.shadowBlur = 0
  ctx.fillStyle = 'rgba(255, 255, 255, 0.3)'
  ctx.fillRect(250, 530, 300, 50)
  ctx.fillStyle = 'white'
  ctx.font = 'bold 18px Inter, sans-serif'
  ctx.fillText('CLICK TO LEARN MORE →', 400, 560)
  
  return canvas.toDataURL()
}

const imageFor = (topic, index) => {
  const seed = encodeURIComponent(`${topic.id}-${topic.title}-${index}`)
  return `https://picsum.photos/seed/${seed}/1200/800`
}

const masonryItems = computed(() => {
  return topics.map((topic, index) => ({
    image: imageFor(topic, index),
    // deterministic fallback generated locally (always available)
    fallback: generateRichCard(topic, topicColors[index % topicColors.length], index),
    title: topic.title,
    description: topic.description
  }))
})

const handleItemClick = (index) => {
  const topic = topics[index]
  console.log('Clicked item:', index, 'Topic:', topic)
  if (topic) {
    router.push({ name: 'note', params: { id: topic.id } })
  }
}

onMounted(() => {
  setTimeout(() => {
    document.querySelectorAll('.fade-in, .fade-in-delay').forEach(el => {
      el.style.opacity = '1'
      el.style.transform = 'translateY(0)'
    })
  }, 100)
})
</script>

<style scoped>
.fade-in,
.fade-in-delay {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.fade-in-delay {
  transition-delay: 0.3s;
}
</style>
