<template>
  <div class="visual-wrapper" ref="wrapperRef">
    <div class="app-window">
      <div class="window-header">
        <div class="dot red"></div>
        <div class="dot yellow"></div>
        <div class="dot green"></div>
        <span class="window-title">制作</span>
      </div>
      <div class="window-body">
        <div class="start-panel" ref="startPanelRef">
          <div class="produce-summary">
            <span class="summary-label">已选择片段</span>
            <span class="summary-count">3</span>
          </div>
          <button class="start-btn" ref="startBtnRef">开始制作</button>
        </div>

        <div class="recording-panel" ref="recordingPanelRef">
          <div class="recording-header">
            <div class="recording-spinner"></div>
            <span class="recording-text" ref="recordingTextRef">正在录制中...</span>
          </div>
          <div class="recording-progress">
            <div class="rec-progress-bar" ref="recBarRef"></div>
          </div>
          <div class="recording-takes">
            <div v-for="(take, i) in takes" :key="i" class="take-row" :ref="el => { if (el) takeRefs[i] = el as HTMLElement }">
              <div class="take-indicator" :ref="el => { if (el) indicatorRefs[i] = el as HTMLElement }">
                <div class="take-spinner-small"></div>
              </div>
              <div class="take-info">
                <span class="take-name">{{ take.name }}</span>
                <span class="take-detail">{{ take.detail }}</span>
              </div>
              <span class="take-status" :ref="el => { if (el) statusRefs[i] = el as HTMLElement }">待录制</span>
            </div>
          </div>
        </div>

        <div class="complete-panel" ref="completePanelRef">
          <div class="complete-badge">录制完成</div>
          <div class="complete-takes">
            <div v-for="(take, i) in takes" :key="i" class="complete-row">
              <span class="complete-icon">✓</span>
              <span class="complete-name">{{ take.name }}</span>
              <span class="complete-dur">{{ take.dur }}</span>
            </div>
          </div>
          <div class="complete-hint">可前往「剪辑」页面合成视频</div>
        </div>
      </div>
    </div>

    <svg ref="cursorRef" class="fake-cursor" width="20" height="20" viewBox="0 0 24 24" fill="none">
      <path d="M5.5 3.21V20.8c0 .45.54.67.85.35l4.86-4.86a.5.5 0 0 1 .35-.15h6.87c.45 0 .67-.54.35-.85L5.85 2.36a.5.5 0 0 0-.35.85z" fill="#fff" stroke="#111" stroke-width="1.5"/>
    </svg>
    <div ref="rippleRef" class="click-ripple"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

const wrapperRef = ref<HTMLElement | null>(null)
const cursorRef = ref<SVGElement | null>(null)
const rippleRef = ref<HTMLElement | null>(null)
const startPanelRef = ref<HTMLElement | null>(null)
const startBtnRef = ref<HTMLElement | null>(null)
const recordingPanelRef = ref<HTMLElement | null>(null)
const recordingTextRef = ref<HTMLElement | null>(null)
const recBarRef = ref<HTMLElement | null>(null)
const completePanelRef = ref<HTMLElement | null>(null)
const takeRefs = ref<HTMLElement[]>([])
const indicatorRefs = ref<HTMLElement[]>([])
const statusRefs = ref<HTMLElement[]>([])

let ctx: ReturnType<typeof gsap.context> | null = null

const takes = [
  { name: 'R14 — s1mple AWP ACE', detail: '击杀者 + 被害者视角', dur: '12.4s' },
  { name: 'R14 — s1mple AWP 穿墙', detail: '击杀者 + 被害者视角', dur: '8.2s' },
  { name: 'R14 — s1mple Deagle HS', detail: '仅击杀者视角', dur: '6.8s' },
]

function getElPos(el: HTMLElement | null | undefined) {
  if (!el || !wrapperRef.value) return { x: 0, y: 0 }
  const wr = wrapperRef.value.getBoundingClientRect()
  const er = el.getBoundingClientRect()
  return { x: er.left - wr.left + er.width / 2, y: er.top - wr.top + er.height / 2 }
}

onMounted(() => {
  if (!wrapperRef.value) return
  const btnPos = getElPos(startBtnRef.value)

  ctx = gsap.context(() => {
    const tl = gsap.timeline({
      repeat: -1,
      repeatDelay: 1.2,
      scrollTrigger: {
        trigger: wrapperRef.value!,
        start: 'top 80%',
        toggleActions: 'play pause resume reset',
      },
    })

    gsap.set(cursorRef.value, { x: btnPos.x + 30, y: -10, autoAlpha: 0 })
    gsap.set(recordingPanelRef.value, { display: 'none', autoAlpha: 0 })
    gsap.set(completePanelRef.value, { display: 'none', autoAlpha: 0, y: 6 })
    gsap.set(recBarRef.value, { width: '0%' })

    tl.to(cursorRef.value, { autoAlpha: 1, duration: 0.2 })
    tl.to(cursorRef.value, { x: btnPos.x, y: btnPos.y, duration: 0.3, ease: 'power2.inOut' })

    // Click start
    tl.to(cursorRef.value, { scale: 0.85, duration: 0.06 })
    tl.set(rippleRef.value, { x: btnPos.x, y: btnPos.y, scale: 0, autoAlpha: 0.6 })
    tl.to(rippleRef.value, { scale: 2.5, autoAlpha: 0, duration: 0.25 })
    tl.to(cursorRef.value, { scale: 1, duration: 0.08 }, '-=0.2')

    // Switch panels
    tl.to(startPanelRef.value, { autoAlpha: 0, duration: 0.15 })
    tl.set(startPanelRef.value, { display: 'none' })
    tl.to(cursorRef.value, { autoAlpha: 0, duration: 0.15 })
    tl.set(recordingPanelRef.value, { display: 'block' })
    tl.to(recordingPanelRef.value, { autoAlpha: 1, duration: 0.2 })

    // Takes
    for (let i = 0; i < 3; i++) {
      tl.call(() => {
        if (indicatorRefs.value[i]) indicatorRefs.value[i].classList.add('active')
        if (statusRefs.value[i]) { statusRefs.value[i].textContent = '录制中...'; statusRefs.value[i].classList.add('recording') }
        if (recordingTextRef.value) recordingTextRef.value.textContent = `正在录制 ${i + 1}/3...`
      }, [], i === 0 ? '+=0.2' : '+=0.1')

      tl.to(recBarRef.value, { width: `${((i + 1) / 3) * 100}%`, duration: 0.5, ease: 'power1.inOut' })

      tl.call(() => {
        if (indicatorRefs.value[i]) { indicatorRefs.value[i].classList.remove('active'); indicatorRefs.value[i].classList.add('done') }
        if (statusRefs.value[i]) { statusRefs.value[i].textContent = '完成'; statusRefs.value[i].classList.remove('recording'); statusRefs.value[i].classList.add('done') }
      })
    }

    // Complete
    tl.call(() => { if (recordingTextRef.value) recordingTextRef.value.textContent = '全部完成!' }, [], '+=0.2')
    tl.to(recordingPanelRef.value, { autoAlpha: 0, duration: 0.2 }, '+=0.3')
    tl.set(recordingPanelRef.value, { display: 'none' })
    tl.set(completePanelRef.value, { display: 'block' })
    tl.to(completePanelRef.value, { autoAlpha: 1, y: 0, duration: 0.3, ease: 'back.out(1.2)' })

    // Reset
    tl.call(() => {
      indicatorRefs.value.forEach(el => { if (el) el.classList.remove('active', 'done') })
      statusRefs.value.forEach(el => { if (el) { el.textContent = '待录制'; el.classList.remove('recording', 'done') } })
      if (recordingTextRef.value) recordingTextRef.value.textContent = '正在录制中...'
    }, [], '+=1.5')
    tl.set(completePanelRef.value, { display: 'none', autoAlpha: 0, y: 6 })
    tl.set(startPanelRef.value, { display: 'block', autoAlpha: 1 })
    tl.set(recBarRef.value, { width: '0%' })
  }, wrapperRef.value)
})

onUnmounted(() => { ctx?.revert() })
</script>

<style scoped>
.visual-wrapper { position: relative; width: 100%; max-width: 400px; }
.app-window { background: #181b19; border: 1px solid #303732; border-radius: 12px; overflow: hidden; box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5); }
.window-header { display: flex; align-items: center; gap: 6px; padding: 10px 14px; border-bottom: 1px solid #303732; }
.dot { width: 10px; height: 10px; border-radius: 50%; }
.dot.red { background: #a94f4f; } .dot.yellow { background: #d09f49; } .dot.green { background: #2f9462; }
.window-title { margin-left: 8px; font-size: 12px; font-weight: 600; color: #8d9890; }
.window-body { padding: 14px; min-height: 200px; }

.produce-summary { display: flex; align-items: center; justify-content: space-between; padding: 12px 14px; border-radius: 8px; background: #1b1f1d; border: 1px solid #303732; margin-bottom: 14px; }
.summary-label { font-size: 12px; color: #8d9890; font-weight: 600; }
.summary-count { font-size: 18px; font-weight: 800; color: #85d3a7; }
.start-btn { width: 100%; padding: 10px; border-radius: 8px; background: #2f9462; color: #fff; border: none; font-size: 13px; font-weight: 700; cursor: pointer; font-family: inherit; }

.recording-panel { display: none; }
.recording-header { display: flex; align-items: center; gap: 10px; margin-bottom: 14px; }
.recording-spinner { width: 16px; height: 16px; border: 2.5px solid transparent; border-top-color: #2f9462; border-right-color: #2f9462; border-radius: 50%; animation: spin 0.8s linear infinite; }
.recording-text { font-size: 12px; font-weight: 700; color: #c9d3cb; }
.recording-progress { height: 4px; background: #252a27; border-radius: 4px; overflow: hidden; margin-bottom: 16px; }
.rec-progress-bar { height: 100%; background: linear-gradient(90deg, #2f9462, #85d3a7); border-radius: 4px; width: 0%; }
.recording-takes { display: flex; flex-direction: column; gap: 8px; }
.take-row { display: flex; align-items: center; gap: 10px; padding: 9px 11px; border-radius: 7px; background: #1b1f1d; border: 1px solid #303732; }
.take-indicator { width: 20px; height: 20px; border-radius: 50%; background: #252a27; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: background 0.3s; }
.take-indicator.active { background: rgba(208, 159, 73, 0.2); }
.take-indicator.active .take-spinner-small { width: 10px; height: 10px; border: 2px solid transparent; border-top-color: #d09f49; border-radius: 50%; animation: spin 0.7s linear infinite; }
.take-indicator.done { background: rgba(47, 148, 98, 0.2); }
.take-indicator.done::after { content: '✓'; font-size: 10px; color: #2f9462; font-weight: 700; }
.take-indicator.done .take-spinner-small { display: none; }
.take-info { flex: 1; display: flex; flex-direction: column; gap: 1px; }
.take-name { font-size: 11px; font-weight: 700; color: #edf1ee; }
.take-detail { font-size: 9px; color: #8d9890; }
.take-status { font-size: 9px; font-weight: 700; color: #8d9890; transition: color 0.3s; }
.take-status.recording { color: #d09f49; }
.take-status.done { color: #2f9462; }

.complete-panel { display: none; }
.complete-badge { display: inline-block; padding: 4px 14px; border-radius: 999px; background: rgba(47, 148, 98, 0.15); border: 1px solid rgba(47, 148, 98, 0.3); color: #2f9462; font-size: 12px; font-weight: 700; margin-bottom: 14px; }
.complete-takes { display: flex; flex-direction: column; gap: 6px; margin-bottom: 12px; }
.complete-row { display: flex; align-items: center; gap: 8px; padding: 8px 10px; border-radius: 6px; background: rgba(47, 148, 98, 0.05); border: 1px solid rgba(47, 148, 98, 0.15); }
.complete-icon { color: #2f9462; font-size: 11px; font-weight: 700; }
.complete-name { flex: 1; font-size: 10px; font-weight: 700; color: #edf1ee; }
.complete-dur { font-size: 9px; color: #8d9890; font-weight: 600; }
.complete-hint { font-size: 10px; color: #8d9890; text-align: center; padding: 6px; }

.fake-cursor { position: absolute; top: 0; left: 0; z-index: 10; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5)); pointer-events: none; will-change: transform; }
.click-ripple { position: absolute; top: 0; left: 0; width: 16px; height: 16px; border-radius: 50%; background: rgba(47, 148, 98, 0.5); transform: scale(0); opacity: 0; pointer-events: none; z-index: 9; }
@keyframes spin { to { transform: rotate(360deg); } }
</style>
