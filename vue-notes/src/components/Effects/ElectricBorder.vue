<template>
  <div class="eb-wrap" :style="wrapperStyle">
    <svg class="eb-defs" aria-hidden="true" focusable="false">
      <defs>
        <filter :id="`${uid}-filter`" x="-50%" y="-50%" width="200%" height="200%">
          <feTurbulence type="fractalNoise" baseFrequency="0.02" numOctaves="2" seed="3" result="noise">
            <animate attributeName="baseFrequency" :dur="`${speed}s`" values="0.02;0.04;0.02" repeatCount="indefinite" />
          </feTurbulence>
          <feDisplacementMap in="SourceGraphic" in2="noise" :scale="chaos" />
        </filter>
      </defs>
    </svg>

    <div class="eb-ring" :style="ringStyle"></div>
    <div class="eb-slot">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  color: { type: String, default: '#46ff7f' },
  speed: { type: Number, default: 3 },
  chaos: { type: Number, default: 12 },
  thickness: { type: Number, default: 3 },
  borderRadius: { type: Number, default: 16 },
  style: { type: Object, default: () => ({}) }
})

const uid = `eb-${Math.random().toString(36).slice(2)}`

const wrapperStyle = computed(() => ({
  '--eb-color': props.color,
  '--eb-speed': `${props.speed}s`,
  '--eb-thickness': `${props.thickness}px`,
  '--eb-radius': `${props.borderRadius}px`,
  '--eb-filter': `url(#${uid}-filter)`,
  ...props.style
}))

const ringStyle = computed(() => ({
  filter: `var(--eb-filter) drop-shadow(0 0 18px ${hexToRgba(props.color, 1)}) drop-shadow(0 0 36px ${hexToRgba(props.color, 0.85)}) drop-shadow(0 0 54px ${hexToRgba(props.color, 0.6)})`
}))

function hexToRgba(hex, a) {
  const h = hex.replace('#', '')
  const bigint = parseInt(h, 16)
  const r = (bigint >> 16) & 255
  const g = (bigint >> 8) & 255
  const b = bigint & 255
  return `rgba(${r}, ${g}, ${b}, ${a})`
}
</script>

<style scoped>
.eb-wrap {
  position: relative;
  border-radius: var(--eb-radius);
}

/* Ring only (masked) */
.eb-ring {
  position: absolute;
  inset: 0;
  border-radius: inherit;
  padding: var(--eb-thickness);
  background: linear-gradient(90deg, var(--eb-color), var(--eb-color));
  -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  mask-composite: exclude;
  pointer-events: none;
}

.eb-wrap::after {
  content: '';
  position: absolute;
  inset: -18px;
  border-radius: inherit;
  background: radial-gradient(closest-side, var(--eb-color), rgba(0,0,0,0));
  filter: blur(28px);
  opacity: 0.7;
  z-index: 0;
  pointer-events: none;
}

.eb-slot {
  position: relative;
  z-index: 1;
}
</style>
