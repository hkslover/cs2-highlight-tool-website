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
          <div class="download-wrapper" ref="downloadWrapperRef">
            <button
              class="btn btn-primary download-btn"
              :class="{ 'is-open': dropdownOpen }"
              @click="toggleDropdown"
            >
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <path d="M12 3v13M5 16l7 7 7-7"/><line x1="3" y1="23" x2="21" y2="23"/>
              </svg>
              <span>下载</span>
              <span v-if="latestVersion" class="version-tag">{{ latestVersion }}</span>
              <svg class="chevron" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M6 9l6 6 6-6"/></svg>
            </button>

            <transition name="dropdown-fade">
              <div v-if="dropdownOpen" class="download-dropdown">
                <div v-if="fetchError" class="download-error">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12" y2="16"/></svg>
                  获取版本信息失败，请稍后重试
                </div>
                <template v-else-if="downloadSources.length">
                  <a
                    v-for="source in downloadSources"
                    :key="source.label"
                    :href="source.url"
                    target="_blank"
                    rel="noopener noreferrer"
                    class="download-option"
                    @click="dropdownOpen = false"
                  >
                    <span class="option-icon" v-html="source.icon"></span>
                    <div class="option-info">
                      <span class="option-label">{{ source.label }}</span>
                      <span class="option-desc">{{ source.desc }}</span>
                    </div>
                    <svg class="option-arrow" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M7 17L17 7M7 7h10v10"/></svg>
                  </a>
                </template>
                <div v-else class="download-loading">
                  <span class="loading-dot"></span>
                  <span class="loading-dot"></span>
                  <span class="loading-dot"></span>
                </div>
              </div>
            </transition>
          </div>

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

// Download dropdown state
const downloadWrapperRef = ref<HTMLElement | null>(null)
const dropdownOpen = ref(false)
const latestVersion = ref('')
const fetchError = ref(false)

interface DownloadSource {
  label: string
  desc: string
  url: string
  icon: string
}
const downloadSources = ref<DownloadSource[]>([])

const ICON_DIRECT = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M13 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V9z"/><polyline points="13 2 13 9 20 9"/></svg>`
const ICON_GITHUB = `<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>`
const ICON_MIRROR = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 014 10 15.3 15.3 0 01-4 10 15.3 15.3 0 01-4-10 15.3 15.3 0 014-10z"/></svg>`

async function fetchDownloadInfo() {
  try {
    const res = await fetch('https://updates.snowblog.xyz/cs2-highlight-tool-v2.json')
    if (!res.ok) throw new Error('fetch failed')
    const data = await res.json()
    const tool = data?.dependencies?.['cs2-highlight-tool']
    if (!tool) throw new Error('missing key')

    latestVersion.value = tool.latest_tag ?? ''
    const asset = tool.latest?.assets?.[0]
    if (!asset) throw new Error('no asset')

    downloadSources.value = [
      {
        label: '镜像加速下载',
        desc: 'gh-proxy镜像',
        url: asset.mirror_url,
        icon: ICON_MIRROR,
      },
      {
        label: '直链下载',
        desc: '',
        url: asset.url.startsWith('http') ? asset.url : `https://${asset.url}`,
        icon: ICON_DIRECT,
      },
      {
        label: 'GitHub Release',
        desc: '',
        url: asset.github_url,
        icon: ICON_GITHUB,
      },
      
    ]
  } catch {
    fetchError.value = true
  }
}

function toggleDropdown() {
  dropdownOpen.value = !dropdownOpen.value
  if (dropdownOpen.value && !downloadSources.value.length && !fetchError.value) {
    fetchDownloadInfo()
  }
}

function handleClickOutside(e: MouseEvent) {
  if (downloadWrapperRef.value && !downloadWrapperRef.value.contains(e.target as Node)) {
    dropdownOpen.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)

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
  document.removeEventListener('click', handleClickOutside)
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
  align-items: center;
}

.hero-actions .btn-ghost {
  gap: 10px;
  padding: 12px 24px;
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

/* Download dropdown */
.download-wrapper {
  position: relative;
}

.download-btn {
  gap: 10px;
}

.download-btn .chevron {
  transition: transform 0.2s ease;
  opacity: 0.7;
  flex-shrink: 0;
}

.download-btn.is-open .chevron {
  transform: rotate(180deg);
}

.version-tag {
  display: inline-flex;
  align-items: center;
  padding: 1px 8px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.15);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.3px;
}

.download-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  min-width: 260px;
  background: #151918;
  border: 1px solid rgba(47, 148, 98, 0.25);
  border-radius: 12px;
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.6), 0 0 0 1px rgba(47, 148, 98, 0.08);
  overflow: hidden;
  z-index: 100;
}

.download-option {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  color: var(--color-text);
  text-decoration: none;
  transition: background 0.15s;
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);
}

.download-option:last-child {
  border-bottom: none;
}

.download-option:hover {
  background: rgba(47, 148, 98, 0.1);
  color: var(--color-text);
}

.download-option:hover .option-arrow {
  transform: translate(2px, -2px);
  stroke: var(--color-accent);
}

.option-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 8px;
  background: rgba(47, 148, 98, 0.12);
  color: var(--color-accent);
  flex-shrink: 0;
}

.option-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
  min-width: 0;
}

.option-label {
  font-size: 13px;
  font-weight: 700;
  color: var(--color-text);
}

.option-desc {
  font-size: 11px;
  color: var(--color-text-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.option-arrow {
  flex-shrink: 0;
  opacity: 0.4;
  transition: transform 0.15s, stroke 0.15s, opacity 0.15s;
}

.download-option:hover .option-arrow {
  opacity: 1;
}

.download-error {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 16px;
  font-size: 12px;
  color: var(--color-text-muted);
}

.download-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 20px;
}

.loading-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--color-primary);
  animation: loadingPulse 1.2s ease-in-out infinite;
}

.loading-dot:nth-child(2) { animation-delay: 0.2s; }
.loading-dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes loadingPulse {
  0%, 100% { opacity: 0.3; transform: scale(0.8); }
  50% { opacity: 1; transform: scale(1); }
}

.dropdown-fade-enter-active {
  animation: dropdownIn 0.18s cubic-bezier(0.16, 1, 0.3, 1);
}

.dropdown-fade-leave-active {
  animation: dropdownIn 0.12s cubic-bezier(0.16, 1, 0.3, 1) reverse;
}

@keyframes dropdownIn {
  from { opacity: 0; transform: translateY(-6px) scale(0.97); }
  to { opacity: 1; transform: translateY(0) scale(1); }
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

  .download-dropdown {
    left: 50%;
    transform: translateX(-50%);
    min-width: 280px;
  }

  .dropdown-fade-enter-active {
    animation: dropdownInMobile 0.18s cubic-bezier(0.16, 1, 0.3, 1);
  }

  .dropdown-fade-leave-active {
    animation: dropdownInMobile 0.12s cubic-bezier(0.16, 1, 0.3, 1) reverse;
  }

  @keyframes dropdownInMobile {
    from { opacity: 0; transform: translateX(-50%) translateY(-6px) scale(0.97); }
    to { opacity: 1; transform: translateX(-50%) translateY(0) scale(1); }
  }
}
</style>
