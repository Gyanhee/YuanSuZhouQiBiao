<template>
  <div v-if="element" class="modal-overlay" @click.self="$emit('close')">
    <div class="hologram-card">
      <button class="close-btn" @click="$emit('close')">&times;</button>

      <!-- 左侧：3D 原子模型 -->
      <div class="atom-side">
        <AtomViewer v-if="element.shells && element.shells.length" :shells="element.shells" :symbol="element.symbol" :name="isEn ? (element.name_en || element.name) : element.name" :nameEn="isEn ? '' : (element.name_en || '')" :lang="lang" />
      </div>

      <!-- 右侧：信息面板 -->
      <div class="info-side">
        <!-- 头部 -->
        <div class="header-row">
          <div class="big-symbol" :style="{ color: categoryColor }">{{ element.symbol }}</div>
          <div class="header-details">
            <div class="big-name">
              <span>{{ isEn ? (element.name_en || element.name) : element.name }}</span>
              <span class="en-name" v-if="!isEn">{{ element.name_en || '' }}</span>
            </div>
            <div class="header-meta">
              <span class="meta-tag" :style="{ borderColor: categoryColor, color: categoryColor }">{{ isEn ? (element.category_en || element.category_ui_name || element.category) : (element.category_ui_name || element.category) }}</span>
              <span class="meta-tag phase">{{ isEn ? (element.phase_en || element.phase || '-') : (element.phase || '-') }}</span>
              <span class="meta-tag block">{{ element.block ? element.block.toUpperCase() + (isEn ? '-block' : '区') : '-' }}</span>
              <span class="header-mass"><span class="mass-label">{{ isEn ? 'Atomic Mass' : '原子质量' }}</span> {{ element.atomic_mass }}</span>
            </div>
          </div>
        </div>

        <!-- 简介 -->
        <div class="summary-box" v-if="element.summary || element.summary_en">
          <p>{{ isEn ? (element.summary_en || element.summary) : element.summary }}</p>
        </div>

        <!-- Tab -->
        <div class="tab-nav">
          <button v-for="t in tabs" :key="t.key" class="tab-btn" :class="{ active: activeTab === t.key }" @click="activeTab = t.key">{{ t.label }}</button>
        </div>

        <div class="tab-content-panel">
          <!-- 基础 -->
          <div v-show="activeTab === 'basic'">
            <div class="info-section">
              <h3>{{ isEn ? 'Position' : '周期表位置' }}</h3>
              <div class="props-grid props-grid-4">
                <div class="prop-item"><label>{{ isEn ? 'Period' : '周期' }}</label><span>{{ isEn ? 'Period ' + element.period : '第 ' + element.period + ' 周期' }}</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Group' : '族' }}</label><span>{{ isEn ? 'Group ' + element.group : '第 ' + element.group + ' 族' }}</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Block' : '区块' }}</label><span>{{ element.block ? element.block.toUpperCase() + (isEn ? '-block' : ' 区') : '-' }}</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Atomic Number' : '原子序数' }}</label><span>{{ element.number }}</span></div>
              </div>
            </div>
            <div class="info-section">
              <h3>{{ isEn ? 'Electron Configuration' : '电子排布' }}</h3>
              <div class="config-box">
                <div class="config-text" v-html="formatConfig(element.electron_configuration)"></div>
                <div class="config-semantic" v-if="element.electron_configuration_semantic" v-html="formatConfig(element.electron_configuration_semantic)"></div>
                <div class="config-shell" v-if="element.shells">{{ (isEn ? 'Electrons per shell: [' : '每层电子数：[') + element.shells.join(', ') + ']' }}</div>
              </div>
            </div>
            <div class="info-section" v-if="element.valence && element.valence.length">
              <h3>{{ isEn ? 'Common Valence' : '常见化合价' }}</h3>
              <div class="valence-tags"><span v-for="v in element.valence" :key="v" class="valence-tag">{{ v }}</span></div>
            </div>
            <div class="info-section" v-if="element.isotopes && element.isotopes.length">
              <h3>{{ isEn ? 'Isotopes' : '同位素' }}</h3>
              <div class="isotope-tags">
                <span v-for="iso in element.isotopes" :key="iso.m" class="isotope-tag" :class="{ 'isotope-stable': iso.s }">{{ element.symbol }}-{{ iso.m }} <span v-if="iso.s" class="stable-badge">{{ isEn ? 'Stable' : '稳定' }}</span></span>
              </div>
            </div>
          </div>

          <!-- 物理 -->
          <div v-show="activeTab === 'physical'">
            <div class="info-section">
              <h3>{{ isEn ? 'Thermodynamic Properties' : '热力学与物理性质' }}</h3>
              <div class="props-grid props-grid-4">
                <div class="prop-item"><label>{{ isEn ? 'Melting Point' : '熔点' }}</label><span>{{ element.melt ?? '-' }} K</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Boiling Point' : '沸点' }}</label><span>{{ element.boil ?? '-' }} K</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Density' : '密度' }}</label><span>{{ element.density ?? '-' }} g/cm³</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Atomic Radius' : '原子半径' }}</label><span>{{ element.radius ?? '-' }} pm</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Molar Heat' : '摩尔热容' }}</label><span>{{ element.molar_heat ?? '-' }}</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Phase' : '相态' }}</label><span>{{ isEn ? (element.phase_en || element.phase || '-') : (element.phase || '-') }}</span></div>
              </div>
            </div>
            <div class="info-section" v-if="element.appearance || element.appearance_en">
              <h3>{{ isEn ? 'Appearance' : '外观' }}</h3>
              <p class="appearance-text">{{ isEn ? (element.appearance_en || element.appearance) : element.appearance }}</p>
            </div>
          </div>

          <!-- 化学 -->
          <div v-show="activeTab === 'chemical'">
            <div class="info-section">
              <h3>{{ isEn ? 'Chemical Properties' : '化学性质' }}</h3>
              <div class="props-grid">
                <div class="prop-item"><label>{{ isEn ? 'Electronegativity (Pauling)' : '电负性 (Pauling)' }}</label><span>{{ element.electronegativity_pauling ?? '-' }}</span></div>
                <div class="prop-item"><label>{{ isEn ? 'Electron Affinity' : '电子亲和能' }}</label><span>{{ element.electron_affinity ?? '-' }} kJ/mol</span></div>
              </div>
            </div>
            <div class="info-section" v-if="element.ionization_energies && element.ionization_energies.length">
              <h3>{{ isEn ? 'Ionization Energy (kJ/mol)' : '电离能 (kJ/mol)' }}</h3>
              <div class="ionization-list">
                <div v-for="(ie, i) in element.ionization_energies.slice(0, 6)" :key="i" class="ionization-item">
                  <span class="ion-label">{{ (isEn ? 'Level ' : '第 ') + (i + 1) + (isEn ? '' : ' 级') }}</span>
                  <span class="ion-value">{{ ie }}</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 历史 -->
          <div v-show="activeTab === 'history'">
            <div class="info-section">
              <h3>{{ isEn ? 'Discovery' : '发现' }}</h3>
              <div class="history-item"><label>{{ isEn ? 'Discovered by' : '发现者' }}</label><span>{{ isEn ? (element.discovered_by_en || element.discovered_by || '-') : (element.discovered_by || element.discovered_by_en || '-') }}</span></div>
              <div v-if="!isEn && element.discovered_by_en && element.discovered_by_en !== element.discovered_by" class="history-item"><label>英文记载</label><span>{{ element.discovered_by_en }}</span></div>
              <div class="history-item"><label>{{ isEn ? 'Named by' : '命名者' }}</label><span>{{ isEn ? (element.named_by_en || element.named_by || '-') : (element.named_by || element.named_by_en || '-') }}</span></div>
            </div>
            <a v-if="element.source" :href="element.source" target="_blank" class="source-link">{{ isEn ? 'Wikipedia Source →' : 'Wikipedia 来源 →' }}</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import AtomViewer from './AtomViewer.vue'

const props = defineProps<{
  element: any
  categoryColor?: string
  lang?: string
}>()

defineEmits<{
  close: []
}>()

const isEn = computed(() => props.lang === 'en')
const activeTab = ref('basic')

const tabs = computed(() => [
  { key: 'basic', label: props.lang === 'en' ? 'Basic' : '基础' },
  { key: 'physical', label: props.lang === 'en' ? 'Physical' : '物理' },
  { key: 'chemical', label: props.lang === 'en' ? 'Chemical' : '化学' },
  { key: 'history', label: props.lang === 'en' ? 'History' : '历史' },
])

function formatConfig(config: string | null): string {
  if (!config) return '-'
  return config.replace(/([spdf])(\d+)/g, '$1<sup>$2</sup>')
}
</script>

<style scoped>
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,.9); backdrop-filter: blur(8px); z-index: 1000; display: flex; justify-content: center; align-items: center; padding: 20px; }
.hologram-card { background: linear-gradient(135deg, #1a1a20, #0a0a10); border: 1px solid rgba(0,243,255,.2); box-shadow: 0 0 60px rgba(0,243,255,.1); border-radius: 16px; display: flex; overflow: hidden; max-width: 950px; width: 100%; max-height: 85vh; position: relative; }
.close-btn { position: absolute; top: 15px; right: 15px; z-index: 10; font-size: 24px; color: #fff; cursor: pointer; width: 36px; height: 36px; background: rgba(0,0,0,.6); border-radius: 50%; display: flex; align-items: center; justify-content: center; border: 1px solid rgba(255,255,255,.1); padding: 0; line-height: 1; transition: .2s; }
.close-btn:hover { background: rgba(255,0,0,.5); transform: rotate(90deg); }
.atom-side { width: 280px; min-width: 280px; background: radial-gradient(circle at center, rgba(0,243,255,.03), transparent 80%); display: flex; align-items: center; justify-content: center; border-right: 1px solid rgba(255,255,255,.05); align-self: stretch; position: relative; }
.info-side { flex: 1; padding: 20px 25px; min-width: 0; overflow: hidden; }
.header-row { display: flex; align-items: flex-start; gap: 15px; margin-bottom: 12px; padding-bottom: 12px; border-bottom: 1px solid rgba(255,255,255,.1); }
.big-symbol { font-size: 52px; font-weight: 700; font-family: Arial; line-height: 1; }
.header-details { flex: 1; }
.big-name { font-size: 22px; color: #fff; font-weight: 700; margin-bottom: 8px; }
.en-name { font-size: 14px; color: #888; margin-left: 8px; font-weight: 400; }
.header-meta { display: flex; gap: 8px; flex-wrap: wrap; align-items: center; }
.meta-tag { padding: 3px 8px; border: 1px solid #444; border-radius: 4px; font-size: 11px; line-height: 1.4; }
.meta-tag.block { background: rgba(0,243,255,.1); color: #00f3ff; font-weight: 700; }
.header-mass { font-size: 22px; font-weight: 800; color: #fff; margin-left: auto; }
.mass-label { font-size: 12px; font-weight: 400; color: #888; margin-right: 6px; }
.summary-box { background: rgba(255,255,255,.02); padding: 12px 15px; border-radius: 8px; margin-bottom: 12px; border-left: 3px solid rgba(0,243,255,.3); }
.summary-box p { font-size: 14px; line-height: 1.7; color: #ddd; }
.tab-nav { display: flex; gap: 4px; background: rgba(0,0,0,.3); padding: 4px; border-radius: 8px; margin-bottom: 15px; }
.tab-btn { flex: 1; background: transparent; border: none; color: #888; padding: 8px 12px; border-radius: 5px; cursor: pointer; font-size: 13px; transition: .3s; font-weight: 500; }
.tab-btn:hover { color: #fff; background: rgba(255,255,255,.05); }
.tab-btn.active { background: #00f3ff; color: #000; font-weight: 700; }
.tab-content-panel { height: 360px; overflow-y: auto; }
.info-section { margin-bottom: 16px; }
.info-section h3 { font-size: 14px; color: #00f3ff; margin-bottom: 10px; text-transform: uppercase; letter-spacing: 1px; border-bottom: 1px solid rgba(0,243,255,.2); padding-bottom: 5px; }
.config-box { background: rgba(0,243,255,.05); padding: 12px 15px; border-radius: 8px; font-family: 'Cascadia Code', 'Fira Code', Consolas, 'Microsoft YaHei', monospace; }
.config-text { font-size: 16px; color: #e0e0e0; word-break: break-all; line-height: 1.6; font-weight: 600; }
.config-text :deep(sup) { font-weight: 700; }
.config-semantic, .config-shell { font-size: 13px; color: #aaa; margin-top: 6px; }
.props-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
.props-grid-4 { grid-template-columns: repeat(4, 1fr); }
.prop-item { background: rgba(255,255,255,.03); padding: 8px 10px; border-radius: 6px; border-left: 3px solid rgba(0,243,255,.3); }
.prop-item label { font-size: 13px; color: #999; display: block; margin-bottom: 4px; }
.prop-item span { font-size: 15px; color: #fff; font-weight: 400; }
.valence-tags { display: flex; flex-wrap: wrap; gap: 8px; }
.valence-tag { background: linear-gradient(135deg, rgba(255,0,85,.2), rgba(255,0,170,.1)); border: 1px solid rgba(255,0,85,.4); padding: 6px 14px; border-radius: 15px; font-size: 16px; color: #f69; font-weight: 600; }
.isotope-tags { display: flex; flex-wrap: wrap; gap: 8px; }
.isotope-tag { background: rgba(0,168,255,.1); border: 1px solid rgba(0,168,255,.3); padding: 5px 12px; border-radius: 4px; font-size: 14px; color: #6cf; display: flex; align-items: center; gap: 4px; }
.stable-badge { font-size: 11px; padding: 1px 5px; border-radius: 3px; }
.isotope-stable { background: rgba(0,255,136,.1); border-color: rgba(0,255,136,.3); color: #6f9; }
.isotope-stable .stable-badge { background: rgba(0,255,136,.15); color: #0f8; }
.appearance-text { font-size: 15px; color: #ccc; line-height: 1.5; }
.ionization-list { display: flex; flex-direction: column; gap: 6px; max-height: 200px; overflow-y: auto; }
.ionization-item { display: flex; justify-content: space-between; padding: 6px 10px; background: rgba(255,255,255,.03); border-radius: 4px; }
.ion-label { font-size: 13px; color: #888; }
.ion-value { font-size: 14px; color: #fff; font-weight: 400; }
.history-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid rgba(255,255,255,.05); }
.history-item label { font-size: 13px; color: #888; }
.history-item span { font-size: 14px; color: #fff; font-weight: 400; }
.source-link { display: inline-flex; align-items: center; gap: 8px; color: #00f3ff; text-decoration: none; font-size: 13px; padding: 8px 12px; background: rgba(0,243,255,.1); border-radius: 6px; margin-top: 15px; transition: .3s; }
.source-link:hover { background: rgba(0,243,255,.2); }
</style>
