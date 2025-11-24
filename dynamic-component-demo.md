# Vue Dynamic Component Demo

- Just a quick demo how to render component dynamically based on some condition
- Including using `<keep-alive>` to preserve component state

## Demo code

```html
<template>
  <div>
    <!-- Only keep FAQ component alive, exclude Dashboard -->
    <keep-alive :include="['FAQ']">
      <component :is="currentComponentComponent" />
    </keep-alive>
    <!-- Or use exclude -->
    <!-- <keep-alive :exclude="['Dashboard']">
      <component :is="currentComponentComponent" />
    </keep-alive> -->
    <!-- Limit maximum cached instances -->
    <!-- <keep-alive :max="5">
      <component :is="currentComponentComponent" />
    </keep-alive> -->
    <button @click="switchComponent('FAQ')">Show FAQ</button>
    <button @click="switchComponent('Dashboard')">Show Dashboard</button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import FAQ from '@/components/FAQ.vue';
import Dashboard from '@/components/Dashboard.vue';

type ComponentName = 'FAQ' | 'Dashboard';

const currentComponentName = ref<ComponentName>('Dashboard');

const components = {
  FAQ,
  Dashboard
} as const;

const currentComponentComponent = computed(() => components[currentComponentName.value]);

function switchComponent(name: ComponentName) {
  currentComponentName.value = name;
}
</script>
```
