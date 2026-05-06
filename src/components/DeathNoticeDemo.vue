<template>
  <div class="death-notice-feed">
    <div
      v-for="item in renderList"
      :key="item.id"
      class="death-notice"
      :style="getItemStyle(item)"
    >
      <span :class="['dn-name', `side-${item.killerSide}`]">{{ item.killer }}</span>
      <img :src="`/cs2/weapon/${item.weapon}.svg`" class="dn-weapon" alt="" />
      <img v-if="item.headshot" src="/cs2/deathnotice/headshot.svg" class="dn-badge" alt="" />
      <img v-if="item.wallbang" src="/cs2/deathnotice/penetrate.svg" class="dn-badge" alt="" />
      <span :class="['dn-name', `side-${item.victimSide}`]">{{ item.victim }}</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

interface KillEvent {
  id: number
  killer: string
  killerSide: 'ct' | 't'
  victim: string
  victimSide: 'ct' | 't'
  weapon: string
  headshot: boolean
  wallbang: boolean
}

interface InternalItem {
  id: number
  data: Omit<KillEvent, 'id'>
  entering: boolean
  leaving: boolean
  frozenSlot: number
}

interface RenderItem extends Omit<KillEvent, 'id'> {
  id: number
  entering: boolean
  leaving: boolean
  slot: number
}

const SLOT_HEIGHT = 44

const kills: Omit<KillEvent, 'id'>[] = [
  { killer: 's1mple', killerSide: 'ct', victim: 'NiKo', victimSide: 't', weapon: 'awp', headshot: true, wallbang: false },
  { killer: 'ZywOo', killerSide: 'ct', victim: 'rain', victimSide: 't', weapon: 'ak47', headshot: true, wallbang: false },
  { killer: 'donk', killerSide: 't', victim: 'ropz', victimSide: 'ct', weapon: 'deagle', headshot: true, wallbang: true },
  { killer: 'm0NESY', killerSide: 'ct', victim: 'electronic', victimSide: 't', weapon: 'awp', headshot: false, wallbang: false },
  { killer: 'b1t', killerSide: 't', victim: 'Twistzz', victimSide: 'ct', weapon: 'm4a1_silencer', headshot: true, wallbang: false },
  { killer: 'frozen', killerSide: 'ct', victim: 'Ax1Le', victimSide: 't', weapon: 'ak47', headshot: false, wallbang: false },
  { killer: 'jL', killerSide: 't', victim: 'huNter', victimSide: 'ct', weapon: 'usp_silencer', headshot: true, wallbang: false },
  { killer: 'broky', killerSide: 'ct', victim: 'Perfecto', victimSide: 't', weapon: 'awp', headshot: true, wallbang: true },
]

const items = ref<InternalItem[]>([])
let nextId = 0
let killIndex = 0
let timer: ReturnType<typeof setInterval> | null = null

const renderList = computed<RenderItem[]>(() => {
  let slot = 0
  return items.value.map(item => ({
    ...item.data,
    id: item.id,
    entering: item.entering,
    leaving: item.leaving,
    slot: item.leaving ? item.frozenSlot : slot++,
  }))
})

function getItemStyle(item: RenderItem): Record<string, string> {
  const y = item.slot * SLOT_HEIGHT
  if (item.entering) {
    return {
      transform: `translateX(60px) translateY(${y}px)`,
      opacity: '0',
    }
  }
  if (item.leaving) {
    return {
      transform: `translateX(-60px) translateY(${y}px)`,
      opacity: '0',
      pointerEvents: 'none',
    }
  }
  return {
    transform: `translateY(${y}px)`,
    opacity: '1',
  }
}

function addKill() {
  const base = kills[killIndex % kills.length]
  killIndex++

  const visible = items.value.filter(i => !i.leaving)
  if (visible.length >= 4) {
    const oldest = visible[0]
    let slot = 0
    for (const it of items.value) {
      if (it === oldest) break
      if (!it.leaving) slot++
    }
    oldest.frozenSlot = slot
    oldest.leaving = true
    setTimeout(() => {
      items.value = items.value.filter(i => i !== oldest)
    }, 500)
  }

  items.value.push({
    id: nextId++,
    data: base,
    entering: true,
    leaving: false,
    frozenSlot: 0,
  })

  const pushed = items.value[items.value.length - 1]
  nextTick(() => {
    requestAnimationFrame(() => {
      requestAnimationFrame(() => {
        pushed.entering = false
      })
    })
  })
}

onMounted(() => {
  addKill()
  timer = setInterval(addKill, 1800)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<style scoped>
.death-notice-feed {
  position: relative;
  min-height: 200px;
  padding: 20px 0;
}

.death-notice {
  position: absolute;
  right: 0;
  top: 0;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 12px;
  border-radius: 6px;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.06);
  font-size: 14px;
  font-weight: 700;
  line-height: 1;
  white-space: nowrap;
  will-change: transform, opacity;
  transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1),
              opacity 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.dn-name {
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.side-ct { color: var(--color-ct); }
.side-t { color: var(--color-t); }

.dn-weapon {
  height: 20px;
  width: auto;
  filter: brightness(0) invert(1);
  opacity: 0.9;
}

.dn-badge {
  height: 16px;
  width: auto;
  flex-shrink: 0;
}
</style>
