<template>
  <div class="relative min-h-screen bg-black text-white overflow-hidden font-sans antialiased">
    <Aurora
      :color-stops="['#00d8ff', '#7c3aed', '#42b883']"
      :speed="0.5"
    />
    
    <router-view v-slot="{ Component }">
      <transition name="fade" mode="out-in">
        <component :is="Component" />
      </transition>
    </router-view>
  </div>
</template>

<script setup>
import Aurora from './components/Backgrounds/Aurora/Aurora.vue'
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
