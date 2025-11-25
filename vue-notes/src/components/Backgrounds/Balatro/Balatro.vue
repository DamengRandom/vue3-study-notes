<template>
  <div ref="containerRef" class="balatro" :style="style"></div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, watch, useTemplateRef } from 'vue'
import { Renderer, Program, Mesh, Triangle } from 'ogl'

interface BalatroProps {
  color1?: string
  color2?: string
  color3?: string
  contrast?: number
  lighting?: number
  spinAmount?: number
  spinSpeed?: number
  spinRotation?: number
  pixelFilter?: number
  isRotate?: boolean
  style?: Record<string, unknown>
}

const props = withDefaults(defineProps<BalatroProps>(), {
  color1: '#DE443B',
  color2: '#006BB4',
  color3: '#162325',
  contrast: 3.5,
  lighting: 0.4,
  spinAmount: 0.25,
  spinSpeed: 7.0,
  spinRotation: -2.0,
  pixelFilter: 745.0,
  isRotate: false,
  style: () => ({})
})

const containerRef = useTemplateRef<HTMLDivElement>('containerRef')

let renderer: Renderer | null = null
let program: Program | null = null
let animateId = 0

const VERT = `#version 300 es
in vec2 position;
void main() {
  gl_Position = vec4(position, 0.0, 1.0);
}`

const FRAG = `#version 300 es
precision highp float;

uniform float uTime;
uniform vec2 uResolution;
uniform vec3 uColor1;
uniform vec3 uColor2;
uniform vec3 uColor3;
uniform float uContrast;
uniform float uLighting;
uniform float uSpinAmount;
uniform float uSpinSpeed;
uniform float uSpinRotation;
uniform float uPixelFilter;
uniform bool uRotate;

out vec4 fragColor;

vec2 rotate(vec2 p, float a) {
  float s = sin(a); float c = cos(a);
  return vec2(c*p.x - s*p.y, s*p.x + c*p.y);
}

void main() {
  vec2 uv = gl_FragCoord.xy / uResolution;
  uv = uv * 2.0 - 1.0; // center

  float t = uTime * 0.001 * uSpinSpeed;
  float rot = radians(uSpinRotation) + (uRotate ? t * 0.3 : 0.0);
  vec2 p = rotate(uv, rot);

  float len = length(p);
  float swirl = sin(len * 6.2831 + t) * uSpinAmount; // swirl field
  p = rotate(p, swirl);

  // pixelation / posterize
  vec2 cell = vec2(uPixelFilter / uResolution.x, uPixelFilter / uResolution.y);
  p = floor(p / cell) * cell;

  // tri-color blend
  float mix1 = smoothstep(-0.6, 0.4, p.x) * uContrast;
  float mix2 = smoothstep(-0.4, 0.6, p.y) * uContrast;
  vec3 base = mix(uColor1, uColor2, clamp(mix1, 0.0, 1.0));
  vec3 col = mix(base, uColor3, clamp(mix2, 0.0, 1.0));

  // lighting
  float light = 0.5 + 0.5 * cos((p.x + p.y) * 6.2831 + t);
  col *= (1.0 + uLighting * light);

  fragColor = vec4(col, 0.85);
}`

const init = () => {
  const container = containerRef.value
  if (!container) return

  renderer = new Renderer({ alpha: true, antialias: true, premultipliedAlpha: true })
  const gl = renderer.gl
  gl.clearColor(0, 0, 0, 0)
  gl.canvas.style.backgroundColor = 'transparent'

  const tri = new Triangle(gl)
  program = new Program(gl, {
    vertex: VERT,
    fragment: FRAG,
    uniforms: {
      uTime: { value: 0 },
      uResolution: { value: [container.offsetWidth || 300, container.offsetHeight || 300] },
      uColor1: { value: hexToRgb(props.color1) },
      uColor2: { value: hexToRgb(props.color2) },
      uColor3: { value: hexToRgb(props.color3) },
      uContrast: { value: props.contrast },
      uLighting: { value: props.lighting },
      uSpinAmount: { value: props.spinAmount },
      uSpinSpeed: { value: props.spinSpeed },
      uSpinRotation: { value: props.spinRotation },
      uPixelFilter: { value: props.pixelFilter },
      uRotate: { value: props.isRotate }
    }
  })

  const mesh = new Mesh(gl, { geometry: tri, program })
  container.appendChild(gl.canvas)
  fit()

  const render = (t: number) => {
    if (!renderer || !program) return
    program.uniforms.uTime.value = t
    renderer.render({ scene: mesh })
    animateId = requestAnimationFrame(render)
  }
  animateId = requestAnimationFrame(render)

  window.addEventListener('resize', fit)
}

const fit = () => {
  const container = containerRef.value
  if (!container || !renderer || !program) return
  const width = Math.max(container.parentElement?.offsetWidth || container.offsetWidth || window.innerWidth, 300)
  const height = Math.max(container.parentElement?.offsetHeight || container.offsetHeight || 200, 200)
  renderer.setSize(width, height)
  program.uniforms.uResolution.value = [width, height]
}

onMounted(init)
onUnmounted(() => {
  cancelAnimationFrame(animateId)
  window.removeEventListener('resize', fit)
})

watch(() => [props.color1, props.color2, props.color3, props.contrast, props.lighting, props.spinAmount, props.spinSpeed, props.spinRotation, props.pixelFilter, props.isRotate], () => {
  if (!program) return
  program.uniforms.uColor1.value = hexToRgb(props.color1)
  program.uniforms.uColor2.value = hexToRgb(props.color2)
  program.uniforms.uColor3.value = hexToRgb(props.color3)
  program.uniforms.uContrast.value = props.contrast
  program.uniforms.uLighting.value = props.lighting
  program.uniforms.uSpinAmount.value = props.spinAmount
  program.uniforms.uSpinSpeed.value = props.spinSpeed
  program.uniforms.uSpinRotation.value = props.spinRotation
  program.uniforms.uPixelFilter.value = props.pixelFilter
  program.uniforms.uRotate.value = props.isRotate
})

function hexToRgb(hex: string): [number, number, number] {
  const h = hex.replace('#', '')
  const bigint = parseInt(h, 16)
  const r = ((bigint >> 16) & 255) / 255
  const g = ((bigint >> 8) & 255) / 255
  const b = (bigint & 255) / 255
  return [r, g, b]
}
</script>

<style scoped>
.balatro {
  position: absolute;
  inset: 0;
  border-radius: 12px;
  overflow: hidden;
  z-index: 0;
}
</style>
