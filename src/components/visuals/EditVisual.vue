<template>
  <div class="visual-wrapper" ref="wrapperRef">
    <div class="app-window">
      <div class="window-header">
        <div class="dot red"></div>
        <div class="dot yellow"></div>
        <div class="dot green"></div>
        <span class="window-title">剪辑</span>
      </div>
      <div class="window-body">
        <div class="edit-layout">
          <div class="panel source-panel">
            <div class="panel-head">
              <span>可用片段</span>
              <span class="count-tag">{{ sourceClips.length }}</span>
            </div>
            <div class="clip-list">
              <div
                v-for="(clip, i) in sourceClips"
                :key="i"
                class="clip-item"
                :class="{ flying: flyingIndex === i }"
                :ref="el => { if (el) sourceRefs[i] = el as HTMLElement }"
              >
                <span class="clip-tag" :class="clip.type">{{ clip.typeLabel }}</span>
                <span class="clip-name">{{ clip.name }}</span>
                <button class="add-btn" :ref="el => { if (el) btnRefs[i] = el as HTMLElement }">添加</button>
              </div>
            </div>
          </div>

          <div class="panel sequence-panel">
            <div class="panel-head">
              <span>剪辑序列</span>
              <span class="count-tag">{{ sequence.length }}</span>
            </div>
            <div class="transition-row">
              <span class="trans-label">转场:</span>
              <span class="trans-mode active">淡入淡出</span>
              <span class="trans-duration">0.5s</span>
            </div>
            <div class="sequence-list" ref="seqListRef">
              <div v-for="(item, i) in sequence" :key="item.id" class="seq-item" :ref="el => { if (el) seqRefs[i] = el as HTMLElement }">
                <span class="seq-num">#{{ i + 1 }}</span>
                <span class="seq-tag" :class="item.type">{{ item.typeLabel }}</span>
                <span class="seq-name">{{ item.name }}</span>
                <span class="seq-dur">{{ item.dur }}</span>
              </div>
            </div>
            <div class="export-row">
              <button class="export-btn" ref="exportBtnRef">合成视频</button>
              <div class="export-progress" ref="exportProgressRef">
                <div class="export-fill" ref="exportFillRef"></div>
              </div>
            </div>
          </div>
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

interface ClipItem { id: number; name: string; type: string; typeLabel: string; dur: string }

const wrapperRef = ref<HTMLElement | null>(null)
const cursorRef = ref<SVGElement | null>(null)
const rippleRef = ref<HTMLElement | null>(null)
const sourceRefs = ref<HTMLElement[]>([])
const btnRefs = ref<HTMLElement[]>([])
const seqRefs = ref<HTMLElement[]>([])
const seqListRef = ref<HTMLElement | null>(null)
const exportBtnRef = ref<HTMLElement | null>(null)
const exportProgressRef = ref<HTMLElement | null>(null)
const exportFillRef = ref<HTMLElement | null>(null)
const flyingIndex = ref(-1)
const sequence = ref<ClipItem[]>([])

let ctx: ReturnType<typeof gsap.context> | null = null
let idCounter = 0

const sourceClips = [
  { name: 'R14 ACE — Killer', type: 'killer', typeLabel: '击杀者' },
  { name: 'R14 ACE — Victim', type: 'victim', typeLabel: '被害者' },
  { name: 'R7 3K — Killer', type: 'killer', typeLabel: '击杀者' },
]

function getElPos(el: HTMLElement | null | undefined) {
  if (!el || !wrapperRef.value) return { x: 0, y: 0 }
  const wr = wrapperRef.value.getBoundingClientRect()
  const er = el.getBoundingClientRect()
  return { x: er.left - wr.left + er.width / 2, y: er.top - wr.top + er.height / 2 }
}

onMounted(() => {
  if (!wrapperRef.value) return
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

    const firstBtnPos = getElPos(btnRefs.value[0])
    gsap.set(cursorRef.value, { x: firstBtnPos.x, y: -10, autoAlpha: 0 })
    gsap.set(exportProgressRef.value, { autoAlpha: 0 })
    gsap.set(exportFillRef.value, { width: '0%' })

    tl.to(cursorRef.value, { autoAlpha: 1, duration: 0.2 })

    for (let i = 0; i < 3; i++) {
      tl.call(() => {
        const pos = getElPos(btnRefs.value[i])
        gsap.to(cursorRef.value, { x: pos.x, y: pos.y, duration: 0.3, ease: 'power2.inOut' })
      }, [], i === 0 ? undefined : '+=0.15')
      tl.to({}, { duration: 0.35 })

      tl.to(cursorRef.value, { scale: 0.85, duration: 0.05 })
      tl.call(() => {
        const pos = getElPos(btnRefs.value[i])
        gsap.set(rippleRef.value, { x: pos.x, y: pos.y, scale: 0, autoAlpha: 0.6 })
        sequence.value.push({
          id: idCounter++,
          name: sourceClips[i].name,
          type: sourceClips[i].type,
          typeLabel: sourceClips[i].typeLabel,
          dur: ['4.2s', '3.8s', '5.1s'][i],
        })
      })
      tl.to(cursorRef.value, { scale: 1, duration: 0.06 })
      tl.to(rippleRef.value, { scale: 3, autoAlpha: 0, duration: 0.2 }, '-=0.05')
    }

    // Export button
    tl.call(() => {
      const pos = getElPos(exportBtnRef.value)
      gsap.to(cursorRef.value, { x: pos.x, y: pos.y, duration: 0.35, ease: 'power2.inOut' })
    }, [], '+=0.3')
    tl.to({}, { duration: 0.4 })

    tl.to(cursorRef.value, { scale: 0.85, duration: 0.05 })
    tl.call(() => {
      const pos = getElPos(exportBtnRef.value)
      gsap.set(rippleRef.value, { x: pos.x, y: pos.y, scale: 0, autoAlpha: 0.6 })
      if (exportBtnRef.value) exportBtnRef.value.classList.add('active')
    })
    tl.to(cursorRef.value, { scale: 1, duration: 0.06 })
    tl.to(rippleRef.value, { scale: 3, autoAlpha: 0, duration: 0.2 }, '-=0.05')

    tl.to(exportProgressRef.value, { autoAlpha: 1, duration: 0.15 })
    tl.to(exportFillRef.value, { width: '100%', duration: 1.2, ease: 'power1.inOut' })

    tl.to(cursorRef.value, { autoAlpha: 0, duration: 0.2 }, '+=0.3')
    tl.call(() => {
      sequence.value = []
      if (exportBtnRef.value) exportBtnRef.value.classList.remove('active')
    }, [], '+=0.3')
    tl.set(exportProgressRef.value, { autoAlpha: 0 })
    tl.set(exportFillRef.value, { width: '0%' })
  }, wrapperRef.value)
})

onUnmounted(() => { ctx?.revert() })
</script>

<style scoped>
.visual-wrapper { position: relative; width: 100%; max-width: 460px; }
.app-window { background: #181b19; border: 1px solid #303732; border-radius: 12px; overflow: hidden; box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5); }
.window-header { display: flex; align-items: center; gap: 6px; padding: 10px 14px; border-bottom: 1px solid #303732; }
.dot { width: 10px; height: 10px; border-radius: 50%; }
.dot.red { background: #a94f4f; } .dot.yellow { background: #d09f49; } .dot.green { background: #2f9462; }
.window-title { margin-left: 8px; font-size: 12px; font-weight: 600; color: #8d9890; }
.window-body { padding: 12px; }

.edit-layout { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.panel { background: #1b1f1d; border: 1px solid #303732; border-radius: 8px; padding: 8px; }
.panel-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; font-size: 10px; font-weight: 700; color: #c9d3cb; }
.count-tag { font-size: 9px; padding: 1px 5px; border-radius: 3px; background: rgba(47, 148, 98, 0.15); color: #85d3a7; }
.clip-list, .sequence-list { display: flex; flex-direction: column; gap: 4px; min-height: 80px; }
.clip-item { display: flex; align-items: center; gap: 4px; padding: 5px 6px; border-radius: 5px; background: rgba(0, 0, 0, 0.3); font-size: 9px; transition: opacity 0.3s; }
.clip-item.flying { opacity: 0.4; }
.clip-tag, .seq-tag { font-size: 8px; font-weight: 700; padding: 1px 4px; border-radius: 3px; flex-shrink: 0; }
.clip-tag.killer, .seq-tag.killer { background: rgba(111, 156, 230, 0.15); color: #6f9ce6; }
.clip-tag.victim, .seq-tag.victim { background: rgba(234, 190, 84, 0.15); color: #eabe54; }
.clip-name, .seq-name { flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; color: #c9d3cb; font-size: 9px; }
.add-btn { font-size: 8px; padding: 2px 6px; border-radius: 3px; background: rgba(47, 148, 98, 0.2); color: #85d3a7; border: 1px solid rgba(47, 148, 98, 0.3); cursor: pointer; font-weight: 600; }
.transition-row { display: flex; align-items: center; gap: 6px; margin-bottom: 6px; font-size: 9px; }
.trans-label { color: #8d9890; }
.trans-mode { padding: 1px 5px; border-radius: 3px; background: rgba(47, 148, 98, 0.15); color: #85d3a7; font-weight: 600; }
.trans-duration { color: #8d9890; }
.seq-item { display: flex; align-items: center; gap: 4px; padding: 5px 6px; border-radius: 5px; background: rgba(47, 148, 98, 0.06); border: 1px solid rgba(47, 148, 98, 0.15); font-size: 9px; animation: fadeInUp 0.2s ease both; }
.seq-num { font-weight: 800; color: #2f9462; font-size: 10px; }
.seq-dur { font-size: 8px; color: #8d9890; flex-shrink: 0; }
.export-row { margin-top: 10px; display: flex; flex-direction: column; gap: 6px; }
.export-btn { width: 100%; padding: 5px 10px; border-radius: 5px; background: #2f9462; color: #fff; border: none; font-size: 10px; font-weight: 700; cursor: pointer; transition: all 0.2s; }
.export-btn.active { box-shadow: 0 0 12px rgba(47, 148, 98, 0.5); }
.export-progress { height: 3px; background: #303732; border-radius: 3px; overflow: hidden; }
.export-fill { height: 100%; background: #2f9462; border-radius: 3px; width: 0%; }

.fake-cursor { position: absolute; top: 0; left: 0; z-index: 10; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5)); pointer-events: none; }
.click-ripple { position: absolute; top: 0; left: 0; width: 16px; height: 16px; border-radius: 50%; background: rgba(47, 148, 98, 0.5); transform: scale(0); opacity: 0; pointer-events: none; z-index: 9; }
@keyframes fadeInUp { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
</style>
