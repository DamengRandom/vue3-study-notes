<!--
  Masonry Grid Layout Component
  Inspired by Vue Bits
-->

<template>
  <div ref="containerRef" class="masonry-container">
    <div 
      v-for="(item, index) in items" 
      :key="index"
      class="masonry-item"
      :style="getItemStyle(index)"
      @click="handleItemClick(index)"
    >
      <div class="masonry-card">
        <div class="card-top">
          <img
            :src="imageSrcs[index]"
            alt=""
            class="card-image"
            loading="lazy"
            decoding="async"
            crossorigin="anonymous"
            @error="onImgError(index)"
            @load="onImgLoad"
          />
        </div>
        <div class="card-bottom">
          <h3 class="card-title">{{ item.title }}</h3>
          <p class="card-description">{{ item.description }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, watch, nextTick } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    required: true
  },
  columns: {
    type: Number,
    default: 3
  },
  gap: {
    type: Number,
    default: 20
  }
})

const emit = defineEmits(['item-click'])

const containerRef = ref(null)
const columnHeights = ref([])
const itemPositions = ref([])
const imageSrcs = ref([])

const columnWidth = computed(() => {
  if (!containerRef.value) return 0
  const containerWidth = containerRef.value.offsetWidth
  const totalGap = props.gap * (props.columns - 1)
  return (containerWidth - totalGap) / props.columns
})

const calculateLayout = async () => {
  await nextTick()
  if (!containerRef.value) return

  containerRef.value.style.setProperty('--column-width', `${columnWidth.value}px`)

  columnHeights.value = new Array(props.columns).fill(0)
  itemPositions.value = []

  const itemEls = containerRef.value.querySelectorAll('.masonry-item .masonry-card')

  props.items.forEach((_, index) => {
    // Find the shortest column
    const shortestColumnIndex = columnHeights.value.indexOf(Math.min(...columnHeights.value))

    // Calculate position
    const left = shortestColumnIndex * (columnWidth.value + props.gap)
    const top = columnHeights.value[shortestColumnIndex]

  const el = itemEls[index]
  const measuredHeight = el?.offsetHeight ?? 260

    itemPositions.value.push({
      top,
      left,
      width: columnWidth.value
    })

    columnHeights.value[shortestColumnIndex] += measuredHeight + props.gap
  })

  const maxHeight = Math.max(...columnHeights.value)
  containerRef.value.style.height = `${maxHeight}px`
}

const rebuildImageSrcs = () => {
  imageSrcs.value = props.items.map((it) => it.image || it.fallback || '')
}

const onImgError = (idx) => {
  const fallback = props.items[idx]?.fallback || ''
  imageSrcs.value[idx] = fallback
  calculateLayout()
}

const onImgLoad = () => {
  calculateLayout()
}

const prefetchImages = () => {
  imageSrcs.value = props.items.map((it) => it.fallback || '')
  props.items.forEach((it, i) => {
    const url = it.image
    if (!url) return
    const img = new Image()
    img.crossOrigin = 'anonymous'
    img.onload = () => {
      imageSrcs.value[i] = url
      calculateLayout()
    }
    img.onerror = () => {
      imageSrcs.value[i] = it.fallback || ''
      calculateLayout()
    }
    img.src = url
  })
}

const getItemStyle = (index) => {
  const pos = itemPositions.value[index]
  if (!pos) return { opacity: 0 }
  
  return {
    position: 'absolute',
    top: `${pos.top}px`,
    left: `${pos.left}px`,
    width: `${pos.width}px`,
    transition: 'all 0.4s cubic-bezier(0.4, 0, 0.2, 1)'
  }
}

const handleItemClick = (index) => {
  emit('item-click', index)
}

let resizeObserver = null

onMounted(async () => {
  await nextTick()
  prefetchImages()
  calculateLayout()

  if (containerRef.value) {
    resizeObserver = new ResizeObserver(() => {
      calculateLayout()
    })
    resizeObserver.observe(containerRef.value)
  }

  window.addEventListener('load', calculateLayout)
})

onUnmounted(() => {
  if (resizeObserver) {
    resizeObserver.disconnect()
  }
  window.removeEventListener('load', calculateLayout)
})

watch(() => props.items, () => {
  prefetchImages()
  calculateLayout()
}, { deep: true })

watch(() => [props.columns, props.gap], () => {
  calculateLayout()
})
</script>

<style scoped>
.masonry-container {
  position: relative;
  width: 100%;
  min-height: 800px;
  padding-bottom: 100px;
}

.masonry-item {
  cursor: pointer;
  opacity: 0;
  animation: fadeInUp 0.6s ease forwards;
  width: var(--column-width);
}

.masonry-item:nth-child(1) { animation-delay: 0.1s; }
.masonry-item:nth-child(2) { animation-delay: 0.2s; }
.masonry-item:nth-child(3) { animation-delay: 0.3s; }
.masonry-item:nth-child(4) { animation-delay: 0.4s; }
.masonry-item:nth-child(5) { animation-delay: 0.5s; }
.masonry-item:nth-child(6) { animation-delay: 0.6s; }
.masonry-item:nth-child(n+7) { animation-delay: 0.7s; }

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.masonry-card {
  position: relative;
  border-radius: 16px;
  overflow: visible;
  background: transparent;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.masonry-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.45);
}

/* Electric border handled by ElectricBorder.vue */

.card-top {
  position: relative;
  height: 280px;
  border-top-left-radius: 16px;
  border-top-right-radius: 16px;
  overflow: hidden;
}

.card-image {
  position: absolute;
  inset: 0;
  border-radius: inherit;
  z-index: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.card-bottom {
  position: relative;
  background: rgba(0,0,0,0.85);
  border-bottom-left-radius: 16px;
  border-bottom-right-radius: 16px;
  padding: 18px 20px 22px 20px;
}

.card-number {
  position: absolute;
  top: -24px;
  right: 16px;
  width: 42px;
  height: 42px;
  background: rgba(16, 16, 16, 0.6);
  backdrop-filter: blur(8px);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 18px;
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.18);
  box-shadow: 0 8px 20px rgba(0,0,0,0.4);
}

.card-title {
  font-size: 20px;
  font-weight: 800;
  color: #fff;
  margin: 0 0 8px 0;
  line-height: 1.25;
}

.card-description {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.8);
  margin: 0;
  line-height: 1.6;
}
</style>
