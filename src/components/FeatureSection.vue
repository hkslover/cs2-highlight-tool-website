<template>
  <section :id="id" class="feature-section" ref="sectionRef">
    <div class="container">
      <div class="feature-grid" :class="{ reversed: reverse }">
        <div class="feature-text">
          <span class="section-badge reveal-el">{{ badge }}</span>
          <h2 class="section-title reveal-el">{{ title }}</h2>
          <p class="section-desc reveal-el">{{ description }}</p>
          <ul v-if="points.length" class="feature-points reveal-el">
            <li v-for="(point, i) in points" :key="i">
              <span class="point-icon">{{ point.icon }}</span>
              <span>{{ point.text }}</span>
            </li>
          </ul>
        </div>
        <div class="feature-visual reveal-el">
          <slot />
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

const props = withDefaults(defineProps<{
  id?: string
  badge: string
  title: string
  description: string
  points?: { icon: string; text: string }[]
  reverse?: boolean
}>(), {
  points: () => [],
})

const sectionRef = ref<HTMLElement | null>(null)
let ctx: ReturnType<typeof gsap.context> | null = null

onMounted(() => {
  if (!sectionRef.value) return
  ctx = gsap.context(() => {
    const els = gsap.utils.toArray<HTMLElement>('.reveal-el')
    gsap.from(els, {
      y: 50,
      autoAlpha: 0,
      duration: 0.8,
      stagger: 0.15,
      ease: 'power3.out',
      scrollTrigger: {
        trigger: sectionRef.value!,
        start: 'top 80%',
        toggleActions: 'play none none reset',
      },
    })
  }, sectionRef.value)
})

onUnmounted(() => {
  ctx?.revert()
})
</script>

<style scoped>
.feature-section {
  min-height: 100vh;
  display: flex;
  align-items: center;
  position: relative;
  padding: 60px 0;
  scroll-snap-align: start;
  scroll-snap-stop: always;
}

.feature-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 80px;
  align-items: center;
}

.feature-grid.reversed {
  direction: rtl;
}

.feature-grid.reversed > * {
  direction: ltr;
}

.feature-points {
  list-style: none;
  margin-top: 24px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.feature-points li {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 15px;
  color: var(--color-text-secondary);
}

.point-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.feature-visual {
  display: flex;
  align-items: center;
  justify-content: center;
}

.reveal-el {
  visibility: hidden;
}

@media (max-width: 768px) {
  .feature-grid {
    grid-template-columns: 1fr;
    gap: 40px;
  }

  .feature-grid.reversed {
    direction: ltr;
  }
}
</style>
