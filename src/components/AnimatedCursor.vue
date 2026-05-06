<template>
  <div class="cursor-container" ref="containerRef">
    <svg
      ref="cursorRef"
      class="animated-cursor"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
    >
      <path
        d="M5.5 3.21V20.8c0 .45.54.67.85.35l4.86-4.86a.5.5 0 0 1 .35-.15h6.87c.45 0 .67-.54.35-.85L5.85 2.36a.5.5 0 0 0-.35.85z"
        fill="#fff"
        stroke="#111"
        stroke-width="1.5"
      />
    </svg>
    <div ref="rippleRef" class="cursor-ripple"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { gsap } from 'gsap'

const props = defineProps<{
  playing?: boolean
}>()

const containerRef = ref<HTMLElement | null>(null)
const cursorRef = ref<SVGElement | null>(null)
const rippleRef = ref<HTMLElement | null>(null)

let ctx: ReturnType<typeof gsap.context> | null = null
const mainTimeline = ref<gsap.core.Timeline | null>(null)

defineExpose({
  getTimeline: () => mainTimeline.value,
  getCursorEl: () => cursorRef.value,
  getRippleEl: () => rippleRef.value,
  getContainer: () => containerRef.value,
})

onMounted(() => {
  if (!containerRef.value) return
  ctx = gsap.context(() => {}, containerRef.value)
})

onUnmounted(() => {
  ctx?.revert()
})

watch(() => props.playing, (val) => {
  if (val) mainTimeline.value?.play()
  else mainTimeline.value?.pause()
})
</script>

<style scoped>
.cursor-container {
  position: absolute;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
}

.animated-cursor {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.4));
  will-change: transform;
}

.cursor-ripple {
  position: absolute;
  top: 0;
  left: 0;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: rgba(47, 148, 98, 0.4);
  transform: scale(0);
  opacity: 0;
  z-index: 9;
  will-change: transform, opacity;
}
</style>
