<template>
  <div class="visual-wrapper" ref="wrapperRef">
    <div class="app-window">
      <div class="window-header">
        <div class="dot red"></div>
        <div class="dot yellow"></div>
        <div class="dot green"></div>
        <span class="window-title">导入方式</span>
      </div>
      <div class="window-body">
        <!-- Phase 1: Method cards -->
        <div class="method-grid" ref="gridRef">
          <div
            v-for="(m, i) in methods"
            :key="m.title"
            class="method-card"
            :class="{ highlighted: highlightIndex === i }"
            :ref="el => { if (el) cardRefs[i] = el as HTMLElement }"
          >
            <div class="method-icon">
              <img v-if="m.iconSrc" :src="m.iconSrc" class="platform-icon" alt="" />
              <span v-else>{{ m.emoji }}</span>
            </div>
            <div class="method-title">{{ m.title }}</div>
            <div class="method-desc">{{ m.desc }}</div>
          </div>
        </div>

        <!-- Phase 2: Match history -->
        <div class="match-panel" ref="matchPanelRef">
          <div class="match-header">
            <img src="/images/platforms/wanmei.ico" class="match-platform-icon" alt="" />
            <span class="match-title">最近战绩</span>
            <span class="match-status ready">已登录</span>
          </div>
          <div class="match-table">
            <div class="match-table-head">
              <span>结束时间</span>
              <span>地图</span>
              <span>K/D/A</span>
              <span>操作</span>
            </div>
            <div
              v-for="(match, i) in matches"
              :key="i"
              class="match-row"
              :ref="el => { if (el) matchRefs[i] = el as HTMLElement }"
            >
              <span class="match-time">{{ match.time }}</span>
              <span class="match-map">{{ match.map }}</span>
              <span class="match-kda">{{ match.kda }}</span>
              <button
                class="match-btn"
                :class="{ active: activeMatchBtn === i, success: successMatch === i }"
                :ref="el => { if (el) matchBtnRefs[i] = el as HTMLElement }"
              >
                {{ successMatch === i ? '已导入' : '下载并导入' }}
              </button>
            </div>
          </div>
        </div>

        <!-- Phase 3: Import progress -->
        <div class="import-progress" ref="progressPanelRef">
          <div class="progress-header">
            <span class="progress-label">正在导入 Demo...</span>
            <span class="progress-pct" ref="pctRef">0%</span>
          </div>
          <div class="progress-track">
            <div class="progress-fill" ref="progressFillRef"></div>
          </div>
        </div>

        <!-- Phase 4: Demo result -->
        <div class="demo-result" ref="resultPanelRef">
          <div class="result-badge">导入成功</div>
          <div class="result-grid">
            <div class="result-item">
              <span class="result-key">地图</span>
              <span class="result-val">de_mirage</span>
            </div>
            <div class="result-item">
              <span class="result-key">比分</span>
              <span class="result-val">16 : 13</span>
            </div>
            <div class="result-item">
              <span class="result-key">时长</span>
              <span class="result-val">42分18秒</span>
            </div>
            <div class="result-item">
              <span class="result-key">回合数</span>
              <span class="result-val">29</span>
            </div>
            <div class="result-item">
              <span class="result-key">玩家数</span>
              <span class="result-val">10</span>
            </div>
            <div class="result-item">
              <span class="result-key">Tick Rate</span>
              <span class="result-val">64</span>
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

const wrapperRef = ref<HTMLElement | null>(null)
const cursorRef = ref<SVGElement | null>(null)
const rippleRef = ref<HTMLElement | null>(null)
const gridRef = ref<HTMLElement | null>(null)
const matchPanelRef = ref<HTMLElement | null>(null)
const progressPanelRef = ref<HTMLElement | null>(null)
const progressFillRef = ref<HTMLElement | null>(null)
const pctRef = ref<HTMLElement | null>(null)
const resultPanelRef = ref<HTMLElement | null>(null)
const cardRefs = ref<HTMLElement[]>([])
const matchRefs = ref<HTMLElement[]>([])
const matchBtnRefs = ref<HTMLElement[]>([])
const highlightIndex = ref(-1)
const activeMatchBtn = ref(-1)
const successMatch = ref(-1)

let ctx: ReturnType<typeof gsap.context> | null = null

const methods = [
  { emoji: '📁', iconSrc: '', title: '文件导入', desc: '从本地选择 .dem 文件' },
  { emoji: '', iconSrc: '/images/platforms/wanmei.ico', title: '完美对战平台', desc: '同步最近战绩并导入' },
  { emoji: '', iconSrc: '/images/platforms/fivee.ico', title: '5E 平台', desc: '批量 Match ID 导入' },
]

const matches = [
  { time: '05-06 12:34', map: 'de_mirage', kda: '28/15/6' },
  { time: '05-06 11:02', map: 'de_inferno', kda: '19/18/4' },
  { time: '05-05 22:17', map: 'de_dust2', kda: '24/12/8' },
]

function getElPos(el: HTMLElement | null | undefined) {
  if (!el || !wrapperRef.value) return { x: 0, y: 0 }
  const wr = wrapperRef.value.getBoundingClientRect()
  const er = el.getBoundingClientRect()
  return {
    x: er.left - wr.left + er.width / 2,
    y: er.top - wr.top + er.height / 2,
  }
}

onMounted(() => {
  if (!wrapperRef.value) return

  const cardPos = getElPos(cardRefs.value[1])

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

    gsap.set(cursorRef.value, { x: cardPos.x + 40, y: -10, autoAlpha: 0 })
    gsap.set(matchPanelRef.value, { autoAlpha: 0, y: 8, display: 'none' })
    gsap.set(progressPanelRef.value, { autoAlpha: 0, display: 'none' })
    gsap.set(progressFillRef.value, { width: '0%' })
    gsap.set(resultPanelRef.value, { autoAlpha: 0, y: 6, display: 'none' })

    tl.to(cursorRef.value, { autoAlpha: 1, duration: 0.2 })

    // Move to card
    tl.to(cursorRef.value, { x: cardPos.x, y: cardPos.y, duration: 0.35, ease: 'power2.inOut' })
    tl.call(() => { highlightIndex.value = 1 })

    // Click
    tl.to(cursorRef.value, { scale: 0.85, duration: 0.06 })
    tl.set(rippleRef.value, { x: cardPos.x, y: cardPos.y, scale: 0, autoAlpha: 0.6 })
    tl.to(rippleRef.value, { scale: 3, autoAlpha: 0, duration: 0.25 })
    tl.to(cursorRef.value, { scale: 1, duration: 0.08 }, '-=0.2')

    // Hide grid, show match history
    tl.to(gridRef.value, { autoAlpha: 0, y: -6, duration: 0.2, ease: 'power2.in' })
    tl.set(gridRef.value, { display: 'none' })
    tl.set(matchPanelRef.value, { display: 'block' })
    tl.to(matchPanelRef.value, { autoAlpha: 1, y: 0, duration: 0.25, ease: 'power2.out' })
    tl.from(matchRefs.value, { autoAlpha: 0, x: -6, duration: 0.2, stagger: 0.06, ease: 'power2.out' }, '-=0.15')

    // Move cursor to button
    tl.call(() => {
      const pos = getElPos(matchBtnRefs.value[0])
      gsap.to(cursorRef.value, { x: pos.x, y: pos.y, duration: 0.3, ease: 'power2.inOut' })
    }, [], '+=0.2')
    tl.to({}, { duration: 0.35 })

    // Click download button
    tl.to(cursorRef.value, { scale: 0.85, duration: 0.06 })
    tl.call(() => {
      const pos = getElPos(matchBtnRefs.value[0])
      gsap.set(rippleRef.value, { x: pos.x, y: pos.y, scale: 0, autoAlpha: 0.6 })
    })
    tl.call(() => { activeMatchBtn.value = 0 })
    tl.to(rippleRef.value, { scale: 3, autoAlpha: 0, duration: 0.25 })
    tl.to(cursorRef.value, { scale: 1, duration: 0.08 }, '-=0.2')

    // Show progress
    tl.to(matchPanelRef.value, { autoAlpha: 0, y: -6, duration: 0.2 }, '+=0.1')
    tl.set(matchPanelRef.value, { display: 'none' })
    tl.set(progressPanelRef.value, { display: 'block' })
    tl.to(progressPanelRef.value, { autoAlpha: 1, duration: 0.2 })
    tl.to(cursorRef.value, { autoAlpha: 0, duration: 0.15 }, '-=0.1')

    // Progress bar
    tl.to(progressFillRef.value, {
      width: '100%',
      duration: 1.0,
      ease: 'power1.inOut',
      onUpdate: function () {
        const progress = this as unknown as gsap.core.Tween
        if (pctRef.value) {
          pctRef.value.textContent = Math.round(progress.progress() * 100) + '%'
        }
      },
    })

    // Show result
    tl.to(progressPanelRef.value, { autoAlpha: 0, duration: 0.15 }, '+=0.1')
    tl.set(progressPanelRef.value, { display: 'none' })
    tl.set(resultPanelRef.value, { display: 'block' })
    tl.to(resultPanelRef.value, { autoAlpha: 1, y: 0, duration: 0.3, ease: 'back.out(1.4)' })

    // Hold then reset
    tl.call(() => {
      highlightIndex.value = -1
      activeMatchBtn.value = -1
      successMatch.value = -1
    }, [], '+=1.5')
    tl.set(resultPanelRef.value, { autoAlpha: 0, y: 6, display: 'none' })
    tl.set(progressFillRef.value, { width: '0%' })
    tl.set(gridRef.value, { display: 'grid', autoAlpha: 1, y: 0 })
    tl.call(() => {
      if (pctRef.value) pctRef.value.textContent = '0%'
    })
  }, wrapperRef.value)
})

onUnmounted(() => {
  ctx?.revert()
})
</script>

<style scoped>
.visual-wrapper {
  position: relative;
  width: 100%;
  max-width: 460px;
}

.app-window {
  background: #181b19;
  border: 1px solid #303732;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

.window-header {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 14px;
  border-bottom: 1px solid #303732;
}

.dot { width: 10px; height: 10px; border-radius: 50%; }
.dot.red { background: #a94f4f; }
.dot.yellow { background: #d09f49; }
.dot.green { background: #2f9462; }

.window-title {
  margin-left: 8px;
  font-size: 12px;
  font-weight: 600;
  color: #8d9890;
}

.window-body {
  padding: 14px;
  min-height: 220px;
}

.method-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
}

.method-card {
  border: 1px solid #303732;
  border-radius: 10px;
  background: linear-gradient(180deg, #1b1f1d 0%, #171a18 100%);
  padding: 10px;
  transition: border-color 0.2s, transform 0.15s, box-shadow 0.2s;
}

.method-card.highlighted {
  border-color: #2f9462;
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(47, 148, 98, 0.25);
}

.method-icon {
  width: 28px;
  height: 28px;
  border-radius: 6px;
  border: 1px solid #2f3631;
  background: rgba(47, 148, 98, 0.08);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
  font-size: 14px;
}

.platform-icon { width: 18px; height: 18px; object-fit: contain; }
.method-title { font-size: 11px; font-weight: 700; color: #edf1ee; }
.method-desc { font-size: 9px; color: #8d9890; margin-top: 3px; line-height: 1.4; }

.match-panel { display: none; }
.match-header { display: flex; align-items: center; gap: 8px; margin-bottom: 12px; padding-bottom: 8px; border-bottom: 1px solid #303732; }
.match-platform-icon { width: 18px; height: 18px; }
.match-title { font-size: 12px; font-weight: 700; color: #edf1ee; flex: 1; }
.match-status { font-size: 9px; font-weight: 700; padding: 2px 8px; border-radius: 999px; }
.match-status.ready { background: rgba(47, 148, 98, 0.15); color: #2f9462; }
.match-table-head { display: grid; grid-template-columns: 1.3fr 1fr 0.8fr 1.2fr; gap: 4px; padding: 5px 8px; font-size: 9px; font-weight: 700; color: #8d9890; border-bottom: 1px solid #252a27; }
.match-row { display: grid; grid-template-columns: 1.3fr 1fr 0.8fr 1.2fr; gap: 4px; padding: 8px 8px; font-size: 10px; color: #c9d3cb; border-bottom: 1px solid #1f2421; align-items: center; }
.match-row:last-child { border-bottom: none; }
.match-time { color: #8d9890; font-size: 9px; }
.match-map { color: #edf1ee; font-weight: 600; }
.match-kda { color: #85d3a7; font-weight: 700; }
.match-btn { font-size: 9px; font-weight: 700; padding: 4px 8px; border-radius: 5px; background: rgba(47, 148, 98, 0.15); color: #85d3a7; border: 1px solid rgba(47, 148, 98, 0.3); cursor: pointer; transition: all 0.2s; white-space: nowrap; }
.match-btn.active { background: #2f9462; color: #fff; border-color: #2f9462; box-shadow: 0 2px 8px rgba(47, 148, 98, 0.4); }
.match-btn.success { background: rgba(47, 148, 98, 0.2); color: #2f9462; border-color: rgba(47, 148, 98, 0.4); }

.import-progress { display: none; }
.progress-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.progress-label { font-size: 12px; font-weight: 600; color: #c9d3cb; }
.progress-pct { font-size: 12px; font-weight: 800; color: #2f9462; }
.progress-track { height: 6px; background: #252a27; border-radius: 6px; overflow: hidden; }
.progress-fill { height: 100%; background: linear-gradient(90deg, #2f9462, #85d3a7); border-radius: 6px; width: 0%; transition: none; }

.demo-result { display: none; }
.result-badge { display: inline-block; padding: 4px 12px; border-radius: 999px; background: rgba(47, 148, 98, 0.15); border: 1px solid rgba(47, 148, 98, 0.3); color: #2f9462; font-size: 11px; font-weight: 700; margin-bottom: 14px; }
.result-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; }
.result-item { padding: 8px 10px; border-radius: 8px; background: #1b1f1d; border: 1px solid #303732; display: flex; flex-direction: column; gap: 3px; }
.result-key { font-size: 9px; color: #8d9890; font-weight: 600; }
.result-val { font-size: 12px; font-weight: 800; color: #edf1ee; }

.fake-cursor { position: absolute; top: 0; left: 0; z-index: 10; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5)); pointer-events: none; will-change: transform; }
.click-ripple { position: absolute; top: 0; left: 0; width: 16px; height: 16px; border-radius: 50%; background: rgba(47, 148, 98, 0.5); transform: scale(0); opacity: 0; pointer-events: none; z-index: 9; }
</style>
