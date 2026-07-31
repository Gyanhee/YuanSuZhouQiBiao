<template>
  <div
    class="element-card"
    :class="{ placeholder: isPlaceholder, dimmed: isDimmed }"
    :style="borderStyle"
    @click="emit('select', { number, symbol, name, category: categoryColor })"
  >
    <template v-if="isPlaceholder">
      <span class="range-num">{{ symbol }}</span>
      <span class="name">{{ name }}</span>
    </template>
    <template v-else>
      <span class="atomic-number">{{ number }}</span>
      <span class="symbol" :style="{ color: hasBackground ? 'inherit' : categoryColor }">{{ symbol }}</span>
      <span class="name">{{ name }}</span>
      <span class="detail-val" v-if="detailVal">{{ detailVal }}</span>
    </template>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  number: number
  symbol: string
  name: string
  isPlaceholder?: boolean
  categoryColor?: string
  isDimmed?: boolean
  detailVal?: string
  hasBackground?: boolean
}>()

const borderStyle = computed(() => {
  const c = props.categoryColor || 'rgba(255, 255, 255, .1)'
  const style: any = { borderColor: c }
  if (props.hasBackground && props.categoryColor) {
    style.backgroundColor = c.replace('rgb', 'rgba').replace(')', ',0.6)')
  }
  return style
})
const emit = defineEmits<{
    select: [el: any]
  }>()
</script>

<style scoped>
.element-card {
  background: rgba(255, 255, 255, .03);
  border: 1.5px solid rgba(255, 255, 255, .1);
  border-radius: 6px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: all .3s ease;
  padding: 4px;
  color: #e0e0e0;
  position: relative;
  width: 68px;
  height: 76px;
}
.element-card:hover {
  transform: scale(1.15);
  z-index: 100;
  box-shadow: 0 0 25px rgba(0, 0, 0, .8) !important;
  border-color: #fff !important;
}
.atomic-number {
  position: absolute;
  top: 4px;
  left: 5px;
  font-size: 11px;
  opacity: .6;
}
.symbol {
  font-size: 25px;
  font-weight: 700;
  text-shadow: 0 1px 3px rgba(0, 0, 0, .5);
  line-height: 1.1;
  margin-top: 4px;
  font-family: Arial, sans-serif;
}
.detail-val { position: absolute; bottom: 3px; right: 5px; font-size: 9px; font-weight: 700; color: #00f3ff; }
.name {
  font-size: 12px;
  opacity: .95;
  font-weight: 500;
}
.placeholder {
  border: 1.5px dashed rgba(255, 255, 255, .15);
  background: rgba(255, 255, 255, .02);
  justify-content: center;
  align-items: center;
  gap: 2px;
}
.placeholder .range-num {
  font-size: 13px;
  color: #00f3ff;
  font-weight: 700;
}
.placeholder .name {
  font-size: 11px;
  opacity: .8;
  margin: 0;
}
.placeholder:hover {
  background: rgba(255, 255, 255, .08);
  border-style: solid;
  transform: scale(1.05) !important;
}
.dimmed {
  opacity: .15 !important;
  border-color: rgba(255, 255, 255, .05) !important;
  box-shadow: none !important;
  pointer-events: none;
}
.placeholder.dimmed {
  opacity: .08 !important;
  border-color: rgba(255, 255, 255, .03) !important;
}
.dimmed .symbol { color: #888 !important; }
.dimmed .detail-val { color: #888 !important; }
</style>
