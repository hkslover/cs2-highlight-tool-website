<template>
  <div class="workflow" ref="wrapperRef">
    <div
      v-for="(step, i) in steps"
      :key="i"
      class="wf-step"
      :class="{
        active: activeIndex === i,
        done: i < activeIndex || allDone,
      }"
    >
      <div class="wf-node">
        <div class="wf-ring">
          <svg v-if="i < activeIndex || allDone" class="wf-check" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M5 13l4 4L19 7"/></svg>
          <span v-else class="wf-num">{{ i + 1 }}</span>
        </div>
        <div v-if="i < steps.length - 1" class="wf-line">
          <div class="wf-line-fill"></div>
        </div>
      </div>
      <div class="wf-body">
        <span class="wf-label">{{ step.label }}</span>
        <span class="wf-desc">{{ step.desc }}</span>
      </div>
    </div>

    <div class="wf-result" :class="{ visible: allDone }">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 11-5.93-9.14"/><path d="M22 4L12 14.01l-3-3"/></svg>
      <span>高光集锦已生成</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'

const wrapperRef = ref<HTMLElement | null>(null)
const activeIndex = ref(-1)
const allDone = ref(false)

let ctx: ReturnType<typeof gsap.context> | null = null

const steps = [
  { label: '导入 Demo', desc: '完美 / 5E / 本地文件' },
  { label: '选择片段', desc: '筛选高光击杀' },
  { label: '制作', desc: '自动录制无需手动操作' },
  { label: '剪辑', desc: '一键导出高光视频' },
]

onMounted(() => {
  if (!wrapperRef.value) return

  ctx = gsap.context(() => {
    const tl = gsap.timeline({ repeat: -1, repeatDelay: 1.5 })

    tl.call(() => { activeIndex.value = -1; allDone.value = false })

    for (let i = 0; i < 4; i++) {
      tl.call(() => { activeIndex.value = i }, [], `+=${i === 0 ? 0.6 : 0.8}`)
    }

    tl.call(() => { activeIndex.value = 4; allDone.value = true }, [], '+=0.8')

    tl.to({}, { duration: 2.5 })

    tl.call(() => { activeIndex.value = -1; allDone.value = false })
  }, wrapperRef.value)
})

onUnmounted(() => { ctx?.revert() })
</script>

<style scoped>
.workflow {
  display: flex;
  flex-direction: column;
  gap: 0;
  position: relative;
  min-width: 260px;
  text-align: left;
}

.wf-step {
  display: flex;
  gap: 14px;
  position: relative;
}

.wf-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex-shrink: 0;
}

.wf-ring {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: 2px solid #303732;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #181b19;
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  position: relative;
  z-index: 1;
}

.wf-num {
  font-size: 12px;
  font-weight: 800;
  color: #8d9890;
  transition: color 0.3s;
}

.wf-check {
  stroke: #2f9462;
  opacity: 0;
  transform: scale(0.5);
  animation: checkIn 0.3s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes checkIn {
  to { opacity: 1; transform: scale(1); }
}

.wf-step.active .wf-ring {
  border-color: #2f9462;
  box-shadow: 0 0 16px rgba(47, 148, 98, 0.35);
  background: rgba(47, 148, 98, 0.1);
}

.wf-step.active .wf-num {
  color: #85d3a7;
}

.wf-step.done .wf-ring {
  border-color: #2f9462;
  background: rgba(47, 148, 98, 0.12);
}

.wf-line {
  width: 2px;
  height: 28px;
  background: #303732;
  position: relative;
  overflow: hidden;
}

.wf-line-fill {
  position: absolute;
  inset: 0;
  background: #2f9462;
  transform: scaleY(0);
  transform-origin: top;
  transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

.wf-step.done .wf-line-fill {
  transform: scaleY(1);
}

.wf-body {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 5px 0 24px;
  opacity: 0.5;
  transition: opacity 0.3s;
}

.wf-step.active .wf-body,
.wf-step.done .wf-body {
  opacity: 1;
}

.wf-label {
  font-size: 15px;
  font-weight: 800;
  color: var(--color-text);
  line-height: 1.3;
}

.wf-desc {
  font-size: 12px;
  color: var(--color-text-muted);
  font-weight: 600;
}

.wf-step.active .wf-label {
  color: #85d3a7;
}

.wf-result {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 14px;
  border-radius: 8px;
  background: rgba(47, 148, 98, 0.1);
  border: 1px solid rgba(47, 148, 98, 0.25);
  font-size: 13px;
  font-weight: 700;
  color: #2f9462;
  opacity: 0;
  transform: translateY(6px);
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  margin-top: 4px;
}

.wf-result.visible {
  opacity: 1;
  transform: translateY(0);
}

.wf-result svg {
  flex-shrink: 0;
  stroke: #2f9462;
}
</style>
