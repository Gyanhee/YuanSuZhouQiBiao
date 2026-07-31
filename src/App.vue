<template>
  <div class="app">
    <!-- 固定顶栏 -->
    <div class="control-panel">
      <div class="title-row">
        <h1 class="main-title">{{ lang === 'en' ? 'Periodic Table' : '元素周期表' }}</h1>
        <div class="mode-group">
          <button
            v-for="m in modes"
            :key="m.key"
            class="mode-btn"
            :class="{ active: selectedMode === m.key }"
            @click="selectedMode = selectedMode === m.key ? '' : m.key"
          >{{ lang === 'en' && m.enLabel ? m.enLabel : m.label }}</button>
        </div>
        <input v-model="searchKeyword" class="search-box" :placeholder="lang === 'en' ? 'Search element...' : '查找元素...'" />
        <div class="lang-toggle">
          <button class="lang-btn" :class="{ active: lang === 'zh' }" @click="lang = 'zh'">中文</button>
          <button class="lang-btn" :class="{ active: lang === 'en' }" @click="lang = 'en'">EN</button>
        </div>
      </div>
      <div class="legend">
        <span
          v-for="cat in categories"
          :key="cat.name"
          class="legend-item"
          :class="{ active: selectedCategory === cat.name }"
          @click="toggleCategory(cat.name)"
        >
          <span class="legend-dot" :style="{ background: cat.color }"></span>
          {{ lang === 'en' ? cat.enName : cat.name }}
        </span>
      </div>
    </div>

    <div class="periodic-table">
      <!-- 镧系锕系占位卡 -->
      <ElementCard
        :number="0"
        symbol="57-71"
        :name="lang === 'en' ? 'Lanthanide' : '镧系'"
        isPlaceholder
        categoryColor="#f783ac"
        :isDimmed="(selectedCategory && selectedCategory !== '镧系') || searchKeyword.trim() !== ''"
        :style="{ gridRow: 6, gridColumn: 3 }"
      />
      <ElementCard
        :number="0"
        symbol="89-103"
        :name="lang === 'en' ? 'Actinide' : '锕系'"
        isPlaceholder
        categoryColor="#e599f7"
        :isDimmed="(selectedCategory && selectedCategory !== '锕系') || searchKeyword.trim() !== ''"
        :style="{ gridRow: 7, gridColumn: 3 }"
      />

      <!-- 118 个元素 -->
      <ElementCard
        v-for="el in elements"
        :key="el.number"
        :number="el.number"
        :symbol="el.symbol"
        :name="lang === 'zh' ? el.name : el.name_en || el.name"
        :categoryColor="selectedMode ? getHeatColor(el) : getCategoryColor(el.category)"
        :detailVal="selectedMode ? getDetailVal(el) : ''"
        :hasBackground="!!selectedMode"
        :isDimmed="isElementDimmed(el)"
        @select="handleSelect"
        :style="{
          gridRow:
            el.number >= 57 && el.number <= 71 ? 9
            : el.number >= 89 && el.number <= 103 ? 10
            : el.period,
          gridColumn:
            el.number >= 57 && el.number <= 71 ? (el.number - 57 + 3)
            : el.number >= 89 && el.number <= 103 ? (el.number - 89 + 3)
            : el.group
        }"
      />
    </div>

    <DetailPanel
      v-if="selectedElement"
      :element="selectedElement"
      :categoryColor="getCategoryColor(selectedElement.category)"
      :lang="lang"
      @close="selectedElement = null"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ElementCard from './components/ElementCard.vue'
import DetailPanel from './components/DetailPanel.vue'
import data from './data/elements.json'

import { computed } from 'vue'

const elements = data.elements
const selectedCategory = ref('')
const selectedMode = ref('')
const lang = ref('zh')
const searchKeyword = ref('')
const selectedElement = ref<any>(null)

const filteredElements = computed(() => {
  const kw = searchKeyword.value.trim().toLowerCase()
  if (!kw) return elements
  return elements.filter((e: any) =>
    e.symbol.toLowerCase().includes(kw) ||
    e.name.includes(kw) ||
    String(e.number) === kw
  )
})
const modes = [
    { key: '', label: '标准', enLabel: 'Standard' },
    { key: 'radius', label: '半径', enLabel: 'Radius' },
    { key: 'electronegativity', label: '电负性', enLabel: 'Electronegativity' },
    { key: 'ionization', label: '电离能', enLabel: 'Ionization' },
    { key: 'melt', label: '熔点', enLabel: 'Melting' },
    { key: 'boil', label: '沸点', enLabel: 'Boiling' },
    { key: 'density', label: '密度', enLabel: 'Density' },
  ]

function isElementDimmed(el: any): boolean {
  const kw = searchKeyword.value.trim().toLowerCase()
  if (kw) {
    const match = el.symbol.toLowerCase().includes(kw) || el.name.includes(kw) || String(el.number) === kw
    if (!match) return true
  }
  if (selectedCategory.value && !categoriesMatch(el.category, selectedCategory.value)) return true
  return false
}

function handleSelect(el: any) {
  // 从完整数据中查找该元素
  selectedElement.value = elements.find((e: any) => e.number === el.number) || el
}

const categories = [
  { name: '碱金属', enName: 'Alkali Metal', color: '#ff6b6b' },
  { name: '碱土金属', enName: 'Alkaline Earth', color: '#ffa94d' },
  { name: '过渡金属', enName: 'Transition Metal', color: '#ffd43b' },
  { name: '后过渡金属', enName: 'Post-transition', color: '#69db7c' },
  { name: '类金属', enName: 'Metalloid', color: '#38d9a9' },
  { name: '非金属', enName: 'Nonmetal', color: '#4dabf7' },
  { name: '稀有气体', enName: 'Noble Gas', color: '#da77f2' },
  { name: '镧系', enName: 'Lanthanide', color: '#f783ac' },
  { name: '锕系', enName: 'Actinide', color: '#e599f7' },
]

function toggleCategory(name: string) {
  selectedCategory.value = selectedCategory.value === name ? '' : name
}

function categoriesMatch(elementCat: string, selected: string): boolean {
  const c = elementCat.toLowerCase()
  const s = selected
  if (s === '碱金属') return c.includes('碱金属') || (c.includes('alkali metal') && !c.includes('alkaline earth'))
  if (s === '碱土金属') return c.includes('碱土') || c.includes('alkaline earth')
  if (s === '后过渡金属') return c.includes('后过渡') || c.includes('post-transition')
  if (s === '过渡金属') return (c.includes('过渡') && !c.includes('后过渡')) || (c.includes('transition metal') && !c.includes('post-transition'))
  if (s === '类金属') return c.includes('类金属') || c.includes('metalloid')
  if (s === '非金属') return c.includes('非金属')
  if (s === '稀有气体') return c.includes('稀有气体') || c.includes('noble')
  if (s === '镧系') return c.includes('镧系') || c.includes('lanthanide')
  if (s === '锕系') return c.includes('锕系') || c.includes('actinide')
  return false
}

function getCategoryColor(cat: string): string {
  const c = cat.toLowerCase()
  if (c.includes('碱金属') || c.includes('alkali metal') && !c.includes('alkaline earth')) return '#ff6b6b'
  if (c.includes('碱土') || c.includes('alkaline earth')) return '#ffa94d'
  // 后过渡必须先于过渡判断，因为 "post-transition" 也包含 "transition"
  if (c.includes('后过渡') || c.includes('post-transition')) return '#69db7c'
  if (c.includes('过渡') || c.includes('transition metal')) return '#ffd43b'
  if (c.includes('类金属') || c.includes('metalloid')) return '#38d9a9'
  if (c.includes('非金属')) return '#4dabf7'
  if (c.includes('卤素') || c.includes('halogen')) return '#74c0fc'
  if (c.includes('稀有气体') || c.includes('noble')) return '#da77f2'
  if (c.includes('镧系') || c.includes('lanthanide')) return '#f783ac'
  if (c.includes('锕系') || c.includes('actinide')) return '#e599f7'
  return '#888'
}

// ── 热力图 ────
const modeFields: Record<string, string> = {
  radius: 'radius',
  electronegativity: 'electronegativity_pauling',
  ionization: 'ionization_energies',
  melt: 'melt',
  boil: 'boil',
  density: 'density',
}

const heatRange = computed(() => {
  const field = modeFields[selectedMode.value]
  if (!field) return { min: 0, max: 1 }
  let vals: number[] = []
  elements.forEach((e: any) => {
    let v = field === 'ionization_energies' ? e[field]?.[0] : e[field]
    if (typeof v === 'number' && !isNaN(v)) vals.push(v)
  })
  return { min: Math.min(...vals), max: Math.max(...vals) }
})

function getHeatColor(el: any): string {
  const field = modeFields[selectedMode.value]
  if (!field) return getCategoryColor(el.category)
  let val = field === 'ionization_energies' ? el[field]?.[0] : el[field]
  if (typeof val !== 'number' || isNaN(val)) return '#888'
  const { min, max } = heatRange.value
  if (max === min) return '#e0e0e0'
  const t = (val - min) / (max - min)  // 0~1
  // 蓝(低) → 绿(中) → 红(高)
  const r = Math.round(t < 0.5 ? 0 : (t - 0.5) * 2 * 255)
  const g = Math.round(t < 0.5 ? t * 2 * 255 : (1 - t) * 2 * 255)
  const b = Math.round((1 - t) * 255)
  return `rgb(${r},${g},${b})`
}

function getDetailVal(el: any): string {
  const field = modeFields[selectedMode.value]
  if (!field) return ''
  let val = field === 'ionization_energies' ? el[field]?.[0] : el[field]
  if (typeof val !== 'number' || isNaN(val)) return '-'
  let s = val >= 1000 ? (val / 1000).toFixed(1) + 'k' : val.toFixed(1)
  return s.replace('.0k', 'k').replace(/\.0$/, '')
}
</script>

<style>
.app {
  background: #050505;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.control-panel {
  position: sticky; top: 0; z-index: 200;
  background: rgba(20, 20, 25, .95); backdrop-filter: blur(10px);
  padding: 12px 20px 8px;
  border-bottom: 1px solid rgba(255,255,255,.1);
  box-shadow: 0 5px 20px rgba(0,0,0,.5);
  width: 100%; display: flex; flex-direction: column; align-items: center; gap: 10px; height: 108px; overflow: hidden;
}
.title-row { display: flex; align-items: center; justify-content: center; gap: 20px; width: 100%; max-width: 1400px; }
.search-box { background: rgba(0,0,0,.3); border: 1.5px solid rgba(255,255,255,.35); color: #fff; padding: 12px 18px; border-radius: 8px; outline: none; font-size: 16px; width: 200px; transition: .3s; flex-shrink: 0; }
.lang-toggle { display: flex; gap: 4px; width: 120px; justify-content: center; }
.lang-btn { background: transparent; border: 1px solid transparent; color: #aaa; padding: 8px 0; border-radius: 5px; cursor: pointer; font-size: 14px; font-weight: 500; transition: .3s; width: 56px; text-align: center; }
.lang-btn:hover { color: #fff; background: rgba(255,255,255,.05); }
.lang-btn.active { background: #00f3ff; color: #000; font-weight: 700; box-shadow: 0 0 10px rgba(0,243,255,.4); }
.search-box:focus { border-color: #00f3ff; box-shadow: 0 0 15px rgba(0,243,255,.3); }
.search-box::placeholder { color: #aaa; }
.main-title {
  font-size: 30px; font-weight: 800; letter-spacing: 4px; width: 300px; text-align: center;
  background: linear-gradient(90deg, #00f3ff, #00ff88);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text;
  white-space: nowrap; margin: 0; flex-shrink: 0;
}
.legend {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px;
}
.legend-item {
  display: flex;
  align-items: center;
  padding: 5px 10px; font-size: 12px; white-space: nowrap;
  border-radius: 15px;
  background: rgba(255, 255, 255, .05);
  cursor: pointer;
  border: 1px solid transparent;
  transition: .3s;
  color: #aaa;
}
.legend-item:hover {
  background: rgba(255, 255, 255, .1);
  color: #fff;
}
.legend-item.active {
  background: rgba(255, 255, 255, .15);
  border-color: #00f3ff;
  color: #fff;
  box-shadow: 0 0 10px rgba(0, 243, 255, .2);
}
.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 3px;
  margin-right: 6px;
}
.mode-group { display: flex; gap: 6px; background: rgba(0,0,0,.3); border-radius: 8px; padding: 6px; flex-shrink: 0; }
.mode-btn { background: transparent; border: 1px solid transparent; color: #ccc; padding: 10px 12px; border-radius: 6px; cursor: pointer; font-size: 14px; font-weight: 500; transition: .3s; white-space: nowrap; }
.mode-btn:hover { color: #fff; background: rgba(255,255,255,.05); }
.mode-btn.active { background: #00f3ff; color: #000; font-weight: 700; box-shadow: 0 0 10px rgba(0,243,255,.4); }
.periodic-table {
  display: grid;
  grid-template-columns: repeat(18, 68px);
  grid-template-rows: repeat(10, 76px);
  gap: 5px;
  padding-top: 15px;
}
</style>
