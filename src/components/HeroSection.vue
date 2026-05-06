<template>
  <section class="hero" ref="heroRef">
    <div class="hero-bg">
      <video
        ref="heroBgVideo"
        class="hero-bg-video"
        autoplay
        muted
        loop
        playsinline
        :poster="`${videosBase}/hero-poster.jpg`"
      >
        <source :src="`${videosBase}/hero-bg.webm`" type="video/webm" />
        <source :src="`${videosBase}/hero-bg.mp4`" type="video/mp4" />
      </video>
      <div class="hero-bg-overlay"></div>
      <div class="hero-grid"></div>
      <div class="hero-glow"></div>
    </div>

    <div class="container hero-content">
      <div class="hero-left">
        <h1 class="hero-title reveal visible">
          <span class="hero-highlight">CS2</span> 高光集锦<br/>一键制作
        </h1>
        <p class="hero-desc reveal visible">
          从 Demo 导入到成片输出，全链路自动化。
        </p>
        <div class="hero-actions reveal visible">
          <a href="https://github.com" target="_blank" class="btn btn-primary">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
            GitHub
          </a>
          <a href="#features" class="btn btn-ghost">
            了解更多
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M7 13l5 5 5-5M7 6l5 5 5-5"/></svg>
          </a>
        </div>
      </div>

      <div class="hero-right">
        <WorkflowSteps />
      </div>
    </div>

    <div class="hero-scroll-hint">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" opacity="0.5"><path d="M12 5v14M19 12l-7 7-7-7"/></svg>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import WorkflowSteps from './WorkflowSteps.vue'

const videosBase = '/videos'
const heroRef = ref<HTMLElement | null>(null)
const heroBgVideo = ref<HTMLVideoElement | null>(null)
let observer: IntersectionObserver | null = null

onMounted(() => {
  if (!heroRef.value || !heroBgVideo.value) return
  const video = heroBgVideo.value

  const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)')
  if (prefersReduced.matches) {
    video.pause()
    return
  }

  observer = new IntersectionObserver(([entry]) => {
    if (entry.isIntersecting) {
      video.play().catch(() => {})
    } else {
      video.pause()
    }
  }, { threshold: 0.1 })
  observer.observe(heroRef.value)
})

onUnmounted(() => {
  observer?.disconnect()
})
</script>

<style scoped>
.hero {
  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  scroll-snap-align: start;
  scroll-snap-stop: always;
}

.hero-bg {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.hero-bg-video {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.55;
  filter: saturate(0.8) brightness(0.85);
}

.hero-bg-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    180deg,
    rgba(10, 11, 10, 0.6) 0%,
    rgba(10, 11, 10, 0.75) 50%,
    rgba(10, 11, 10, 0.95) 100%
  );
}

@media (prefers-reduced-motion: reduce) {
  .hero-bg-video {
    display: none;
  }
}

.hero-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(47, 148, 98, 0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(47, 148, 98, 0.03) 1px, transparent 1px);
  background-size: 60px 60px;
}

.hero-glow {
  position: absolute;
  top: -30%;
  left: 50%;
  transform: translateX(-50%);
  width: 800px;
  height: 800px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(47, 148, 98, 0.12) 0%, transparent 70%);
  filter: blur(60px);
}

.hero-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 60px;
  padding-top: 80px;
  padding-bottom: 80px;
}

.hero-badge {
  display: inline-block;
  padding: 4px 14px;
  border-radius: 999px;
  background: rgba(47, 148, 98, 0.12);
  border: 1px solid rgba(47, 148, 98, 0.3);
  color: var(--color-accent);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 1.5px;
  margin-bottom: 20px;
}

.hero-title {
  font-size: clamp(40px, 6vw, 64px);
  font-weight: 900;
  line-height: 1.1;
  letter-spacing: -1px;
  margin-bottom: 20px;
}

.hero-highlight {
  background: linear-gradient(135deg, var(--color-primary-glow), var(--color-accent));
  background-size: 200% 200%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: gradientShift 4s ease infinite;
}

.hero-desc {
  font-size: 18px;
  color: var(--color-text-secondary);
  line-height: 1.8;
  margin-bottom: 32px;
}

.hero-actions {
  display: flex;
  gap: 14px;
  flex-wrap: wrap;
}

.hero-right {
  display: flex;
  justify-content: flex-end;
  animation: fadeInUp 1s 0.3s both;
}

.hero-scroll-hint {
  position: absolute;
  bottom: 32px;
  left: 50%;
  transform: translateX(-50%);
  animation: float 3s ease-in-out infinite;
}

@media (max-width: 768px) {
  .hero-content {
    grid-template-columns: 1fr;
    text-align: center;
    gap: 40px;
    padding-top: 100px;
  }

  .hero-right {
    justify-content: center;
  }

  .hero-actions {
    justify-content: center;
  }
}
</style>
