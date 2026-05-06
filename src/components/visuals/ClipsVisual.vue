<template>
  <div class="visual-wrapper" ref="wrapperRef">
    <div class="app-window">
      <div class="window-header">
        <div class="dot red"></div>
        <div class="dot yellow"></div>
        <div class="dot green"></div>
        <span class="window-title">选择片段</span>
      </div>
      <div class="window-body">
        <div class="player-select" ref="selectRef">
          <div class="select-box" :class="{ open: selectOpen }" ref="selectBoxRef">
            <span class="select-value">{{ selectedPlayer }}</span>
            <svg class="select-arrow" width="10" height="10" viewBox="0 0 10 10"><path d="M2 3.5l3 3 3-3" stroke="currentColor" stroke-width="1.5" fill="none"/></svg>
          </div>
          <div class="select-dropdown" v-if="selectOpen" ref="dropdownRef">
            <div
              v-for="(p, i) in players"
              :key="i"
              class="select-option"
              :class="{ active: p.name === selectedPlayer }"
              :ref="el => { if (el) optionRefs[i] = el as HTMLElement }"
            >
              {{ p.name }} ({{ p.kills }})
            </div>
          </div>
        </div>

        <div class="rounds-area" ref="roundsRef">
          <div class="round-block">
            <div class="round-title">第 14 回合 (3 杀)</div>
            <div
              v-for="(kill, i) in kills"
              :key="i"
              class="kill-row"
              :class="{ selected: selectedKills.includes(i), expanded: expandedKill === i }"
              :ref="el => { if (el) killRowRefs[i] = el as HTMLElement }"
            >
              <div class="kill-main">
                <div class="kill-notice">
                  <span class="kn-name ct">{{ kill.killer }}</span>
                  <img :src="`/cs2/weapon/${kill.weapon}.svg`" class="kn-weapon" alt="" />
                  <img v-if="kill.hs" src="/cs2/deathnotice/headshot.svg" class="kn-badge" alt="" />
                  <img v-if="kill.wallbang" src="/cs2/deathnotice/penetrate.svg" class="kn-badge" alt="" />
                  <span class="kn-name t">{{ kill.victim }}</span>
                </div>
                <div class="kill-tags" v-if="selectedKills.includes(i)">
                  <span class="kill-tag green">击杀者视角</span>
                  <span class="kill-tag yellow" v-if="kill.hasVictim">被害者视角</span>
                  <button class="expand-btn" :ref="el => { if (el) expandBtnRefs[i] = el as HTMLElement }">
                    {{ expandedKill === i ? '收起 ▾' : '展开 ▸' }}
                  </button>
                </div>
              </div>
              <div class="kill-settings" v-if="expandedKill === i">
                <div class="setting-row"><span class="setting-label">包含被害者视角</span><div class="setting-switch on"></div></div>
                <div class="setting-row"><span class="setting-label">击杀前置秒数</span><div class="setting-input">5.0</div></div>
                <div class="setting-row"><span class="setting-label">击杀后置秒数</span><div class="setting-input">5.0</div></div>
                <div class="setting-row"><span class="setting-label">启用语音</span><div class="setting-switch on"></div></div>
              </div>
            </div>
          </div>
          <div class="round-block">
            <div class="round-title">第 21 回合 (4 杀)</div>
            <div class="kill-row faded">
              <div class="kill-main">
                <div class="kill-notice">
                  <span class="kn-name ct">s1mple</span>
                  <img src="/cs2/weapon/deagle.svg" class="kn-weapon" alt="" />
                  <img src="/cs2/deathnotice/headshot.svg" class="kn-badge" alt="" />
                  <span class="kn-name t">huNter</span>
                </div>
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
    <div ref="ripple2Ref" class="click-ripple"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

const wrapperRef = ref<HTMLElement | null>(null)
const cursorRef = ref<SVGElement | null>(null)
const rippleRef = ref<HTMLElement | null>(null)
const ripple2Ref = ref<HTMLElement | null>(null)
const selectRef = ref<HTMLElement | null>(null)
const selectBoxRef = ref<HTMLElement | null>(null)
const dropdownRef = ref<HTMLElement | null>(null)
const roundsRef = ref<HTMLElement | null>(null)
const killRowRefs = ref<HTMLElement[]>([])
const expandBtnRefs = ref<HTMLElement[]>([])
const optionRefs = ref<HTMLElement[]>([])

const selectOpen = ref(false)
const selectedPlayer = ref('请选择玩家')
const selectedKills = ref<number[]>([])
const expandedKill = ref(-1)

let ctx: ReturnType<typeof gsap.context> | null = null

const players = [
  { name: 's1mple', kills: 28 },
  { name: 'ZywOo', kills: 19 },
  { name: 'donk', kills: 22 },
]

const kills = [
  { killer: 's1mple', weapon: 'awp', hs: true, wallbang: false, victim: 'NiKo', hasVictim: true },
  { killer: 's1mple', weapon: 'awp', hs: false, wallbang: true, victim: 'rain', hasVictim: true },
  { killer: 's1mple', weapon: 'deagle', hs: true, wallbang: false, victim: 'Ax1Le', hasVictim: false },
]

function getElPos(el: HTMLElement | null | undefined) {
  if (!el || !wrapperRef.value) return { x: 0, y: 0 }
  const wr = wrapperRef.value.getBoundingClientRect()
  const er = el.getBoundingClientRect()
  return { x: er.left - wr.left + er.width / 2, y: er.top - wr.top + er.height / 2 }
}

function doDoubleClickRipple(tl: gsap.core.Timeline, xFn: () => number, yFn: () => number) {
  tl.call(() => { gsap.set(rippleRef.value, { x: xFn(), y: yFn(), scale: 0, autoAlpha: 0.7 }) })
  tl.to(cursorRef.value, { scale: 0.85, duration: 0.04 })
  tl.to(rippleRef.value, { scale: 2.5, autoAlpha: 0, duration: 0.2 })
  tl.to(cursorRef.value, { scale: 1, duration: 0.05 }, '-=0.15')
  tl.call(() => { gsap.set(ripple2Ref.value, { x: xFn(), y: yFn(), scale: 0, autoAlpha: 0.7 }) }, [], '-=0.05')
  tl.to(cursorRef.value, { scale: 0.85, duration: 0.04 }, '-=0.02')
  tl.to(ripple2Ref.value, { scale: 2.5, autoAlpha: 0, duration: 0.2 })
  tl.to(cursorRef.value, { scale: 1, duration: 0.05 }, '-=0.15')
}

onMounted(() => {
  if (!wrapperRef.value) return

  const selectBoxPos = getElPos(selectBoxRef.value)

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

    gsap.set(cursorRef.value, { x: selectBoxPos.x + 60, y: -10, autoAlpha: 0 })
    gsap.set(roundsRef.value, { autoAlpha: 0 })

    tl.to(cursorRef.value, { autoAlpha: 1, duration: 0.2 })

    // Move to select box
    tl.to(cursorRef.value, { x: selectBoxPos.x, y: selectBoxPos.y, duration: 0.3, ease: 'power2.inOut' })

    // Click to open
    tl.to(cursorRef.value, { scale: 0.85, duration: 0.05 })
    tl.to(cursorRef.value, { scale: 1, duration: 0.06 })
    tl.call(() => { selectOpen.value = true }, [], '+=0.05')

    // Move to first option
    tl.call(async () => {
      await nextTick()
      const optPos = getElPos(optionRefs.value[0])
      gsap.to(cursorRef.value, { x: optPos.x, y: optPos.y, duration: 0.2, ease: 'power2.inOut' })
    }, [], '+=0.1')
    tl.to({}, { duration: 0.25 })

    // Click option
    tl.to(cursorRef.value, { scale: 0.85, duration: 0.05 })
    tl.to(cursorRef.value, { scale: 1, duration: 0.06 })
    tl.call(() => { selectedPlayer.value = 's1mple (28)'; selectOpen.value = false })

    // Show rounds
    tl.to(roundsRef.value, { autoAlpha: 1, duration: 0.25, ease: 'power2.out' }, '+=0.1')

    // Double-click first kill row
    tl.call(() => {
      const pos = getElPos(killRowRefs.value[0])
      gsap.to(cursorRef.value, { x: pos.x, y: pos.y, duration: 0.3, ease: 'power2.inOut' })
    }, [], '+=0.15')
    tl.to({}, { duration: 0.35 })
    doDoubleClickRipple(tl, () => getElPos(killRowRefs.value[0]).x, () => getElPos(killRowRefs.value[0]).y)
    tl.call(() => { selectedKills.value.push(0) })

    // Double-click second kill row
    tl.call(() => {
      const pos = getElPos(killRowRefs.value[1])
      gsap.to(cursorRef.value, { x: pos.x, y: pos.y, duration: 0.25, ease: 'power2.inOut' })
    }, [], '+=0.2')
    tl.to({}, { duration: 0.3 })
    doDoubleClickRipple(tl, () => getElPos(killRowRefs.value[1]).x, () => getElPos(killRowRefs.value[1]).y)
    tl.call(() => { selectedKills.value.push(1) })

    // Click expand button
    tl.call(async () => {
      await nextTick()
      const pos = getElPos(expandBtnRefs.value[0])
      gsap.to(cursorRef.value, { x: pos.x, y: pos.y, duration: 0.3, ease: 'power2.inOut' })
    }, [], '+=0.3')
    tl.to({}, { duration: 0.35 })

    tl.to(cursorRef.value, { scale: 0.85, duration: 0.05 })
    tl.call(() => {
      const pos = getElPos(expandBtnRefs.value[0])
      gsap.set(rippleRef.value, { x: pos.x, y: pos.y, scale: 0, autoAlpha: 0.6 })
    })
    tl.to(rippleRef.value, { scale: 2, autoAlpha: 0, duration: 0.2 })
    tl.to(cursorRef.value, { scale: 1, duration: 0.06 }, '-=0.15')
    tl.call(() => { expandedKill.value = 0 })

    // Hold settings visible
    tl.to({}, { duration: 0.8 })

    // Cursor exits
    tl.to(cursorRef.value, { autoAlpha: 0, duration: 0.25 })

    // Reset
    tl.call(() => {
      selectedKills.value = []
      expandedKill.value = -1
      selectedPlayer.value = '请选择玩家'
      selectOpen.value = false
    })
    tl.set(roundsRef.value, { autoAlpha: 0 })
  }, wrapperRef.value)
})

onUnmounted(() => { ctx?.revert() })
</script>

<style scoped>
.visual-wrapper { position: relative; width: 100%; max-width: 440px; }
.app-window { background: #181b19; border: 1px solid #303732; border-radius: 12px; overflow: hidden; box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5); }
.window-header { display: flex; align-items: center; gap: 6px; padding: 10px 14px; border-bottom: 1px solid #303732; }
.dot { width: 10px; height: 10px; border-radius: 50%; }
.dot.red { background: #a94f4f; } .dot.yellow { background: #d09f49; } .dot.green { background: #2f9462; }
.window-title { margin-left: 8px; font-size: 12px; font-weight: 600; color: #8d9890; }
.window-body { padding: 12px; min-height: 280px; }

.player-select { position: relative; margin-bottom: 12px; }
.select-box { display: flex; align-items: center; justify-content: space-between; padding: 7px 10px; border-radius: 6px; border: 1px solid #303732; background: #1b1f1d; cursor: pointer; transition: border-color 0.2s; }
.select-box.open { border-color: #2f9462; }
.select-value { font-size: 11px; font-weight: 600; color: #c9d3cb; }
.select-arrow { color: #8d9890; flex-shrink: 0; }
.select-dropdown { position: absolute; top: 100%; left: 0; right: 0; margin-top: 4px; background: #1b1f1d; border: 1px solid #303732; border-radius: 6px; overflow: hidden; z-index: 5; box-shadow: 0 8px 24px rgba(0,0,0,0.4); }
.select-option { padding: 7px 10px; font-size: 11px; color: #c9d3cb; cursor: pointer; transition: background 0.15s; }
.select-option:hover, .select-option.active { background: rgba(47, 148, 98, 0.12); color: #85d3a7; }

.rounds-area { display: flex; flex-direction: column; gap: 10px; }
.round-title { font-size: 10px; font-weight: 700; color: #8d9890; text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 6px; padding-left: 2px; }
.kill-row { border-radius: 7px; background: rgba(0, 0, 0, 0.3); margin-bottom: 5px; transition: background 0.2s, border-color 0.2s; border: 1px solid transparent; overflow: hidden; }
.kill-row.selected { background: rgba(47, 148, 98, 0.08); border-color: rgba(47, 148, 98, 0.25); }
.kill-row.faded { opacity: 0.45; }
.kill-main { padding: 7px 9px; }
.kill-notice { display: flex; align-items: center; gap: 5px; font-size: 11px; font-weight: 700; }
.kn-name.ct { color: #6f9ce6; } .kn-name.t { color: #eabe54; }
.kn-weapon { height: 15px; width: auto; filter: brightness(0) invert(1); opacity: 0.85; }
.kn-badge { height: 12px; width: auto; flex-shrink: 0; }
.kill-tags { display: flex; align-items: center; gap: 5px; margin-top: 5px; }
.kill-tag { font-size: 8px; font-weight: 700; padding: 2px 6px; border-radius: 3px; }
.kill-tag.green { background: rgba(47, 148, 98, 0.15); color: #2f9462; }
.kill-tag.yellow { background: rgba(234, 190, 84, 0.15); color: #eabe54; }
.expand-btn { margin-left: auto; font-size: 9px; color: #85d3a7; background: none; border: none; cursor: pointer; font-weight: 600; padding: 2px 4px; }

.kill-settings { padding: 8px 10px; border-top: 1px solid #303732; background: rgba(0,0,0,0.2); display: flex; flex-direction: column; gap: 6px; animation: slideDown 0.2s ease both; }
@keyframes slideDown { from { opacity: 0; max-height: 0; } to { opacity: 1; max-height: 200px; } }
.setting-row { display: flex; align-items: center; justify-content: space-between; font-size: 10px; }
.setting-label { color: #8d9890; font-weight: 600; }
.setting-switch { width: 26px; height: 14px; border-radius: 14px; background: #303732; position: relative; }
.setting-switch.on { background: #2f9462; }
.setting-switch::after { content: ''; position: absolute; top: 2px; left: 2px; width: 10px; height: 10px; border-radius: 50%; background: #fff; transition: transform 0.2s; }
.setting-switch.on::after { transform: translateX(12px); }
.setting-input { padding: 2px 8px; border-radius: 4px; background: #252a27; border: 1px solid #303732; color: #edf1ee; font-size: 10px; font-weight: 700; min-width: 40px; text-align: center; }

.fake-cursor { position: absolute; top: 0; left: 0; z-index: 10; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5)); pointer-events: none; will-change: transform; }
.click-ripple { position: absolute; top: 0; left: 0; width: 18px; height: 18px; border-radius: 50%; background: rgba(47, 148, 98, 0.5); transform: scale(0); opacity: 0; pointer-events: none; z-index: 9; }
</style>
