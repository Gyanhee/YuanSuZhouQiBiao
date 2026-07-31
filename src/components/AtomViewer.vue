<template>
  <div class="atom-outer">
    <button class="expand-btn" @click="expanded = true" title="放大查看">⛶</button>
    <div
      class="atom-wrapper"
      ref="wrapper"
      @mousedown.stop="startDrag"
      @click.stop
      @wheel.prevent="onWheel"
    >
      <div class="atom-scene" :style="sceneStyle">
        <div class="nucleus"></div>
        <div class="nucleus-ring"></div>
        <div v-for="(count, i) in shells" :key="i" class="orbit-ring" :style="ringStyle(i)">
          <div class="orbit-spinner" :style="spinnerStyle(i)">
            <div v-for="j in count" :key="j" class="electron" :class="i === shells.length - 1 ? 'valence' : 'inner'" :style="electronStyle(i, count, j)"></div>
          </div>
        </div>
      </div>
      <div class="atom-hint">{{ lang === 'en' ? 'Drag to rotate · Scroll to zoom' : '拖拽旋转 · 滚轮缩放' }}</div>
    </div>
  </div>

  <!-- 全屏 3D -->
  <Teleport to="body">
    <div v-if="expanded" class="expanded-overlay" @click.self="expanded = false">
      <div class="expanded-card"
          @mousedown.stop="startDrag"
          @click.stop
          @wheel.prevent="onExpandedWheel">
        <button class="expanded-close" @mousedown.stop @click="expanded = false">&times;</button>
        <!-- 头部 -->
        <div class="expanded-header">
          <span class="expanded-symbol">{{ symbol }}</span>
          <span class="expanded-name">{{ name }}</span>
          <span class="expanded-en">{{ nameEn }}</span>
        </div>
        <!-- 原子模型 -->
        <div class="expanded-scene" :style="expandedSceneStyle">
          <div class="nucleus"></div>
          <div class="nucleus-ring"></div>
          <div v-for="(count, i) in shells" :key="i" class="orbit-ring" :style="ringStyle(i)">
            <div class="orbit-spinner" :style="spinnerStyle(i)">
              <div v-for="j in count" :key="j" class="electron" :class="i === shells.length - 1 ? 'valence' : 'inner'" :style="electronStyle(i, count, j)"></div>
            </div>
          </div>
        </div>
        <!-- 底部图例 -->
        <div class="expanded-hint">{{ lang === 'en' ? 'Drag to rotate · Scroll to zoom' : '拖拽旋转 · 滚轮缩放' }}</div>
        <div class="expanded-legend">
          <div v-for="(count, i) in shells" :key="i" class="expanded-shell-item" :class="{ valence: i === shells.length - 1 }">
            <span class="shell-dot" :class="i === shells.length - 1 ? 'valence' : 'inner'"></span>
            {{ lang === 'en' ? ('Shell ' + (i+1) + ': ' + count + ' electron' + (count > 1 ? 's' : '') + (i === shells.length - 1 ? ' (valence)' : '')) : ('第' + (i+1) + '层：' + count + ' 个电子' + (i === shells.length - 1 ? '（价电子层）' : '')) }}
          </div>
        </div>
      </div>
    </div>
    </Teleport>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const props = defineProps<{ shells: number[]; symbol?: string; name?: string; nameEn?: string; lang?: string }>()

// 基于电子总数生成不同初始角度，让每个元素打开时视角不同
// 基于电子数给每个元素不同的初始 Z 轴角度（不压扁核）
const zSeed = props.shells.reduce((a, b) => a + b, 0) * 37 % 360
const initZ = zSeed

const rotX = ref(0)
const rotY = ref(0)
const rotZ = ref(initZ)
const zoom = ref(1)
const expandedZoom = ref(1)
const expanded = ref(false)
let dragging = false, prevX = 0, prevY = 0

let targetX = 0, targetY = 0
function startDrag(e: MouseEvent) {
  dragging = true; prevX = e.clientX; prevY = e.clientY
  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', endDrag)
}
function onDrag(e: MouseEvent) {
  if (!dragging) return
  targetX += (e.clientX - prevX) * 0.4
  targetY += (e.clientY - prevY) * 0.4
  prevX = e.clientX; prevY = e.clientY
  requestAnimationFrame(() => {
    rotX.value = targetX
    rotY.value = targetY
  })
}
function endDrag(e: MouseEvent) {
  e.stopPropagation(); e.preventDefault()
  dragging = false; targetX = rotX.value; targetY = rotY.value
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', endDrag)
  // 拖拽结束后短暂阻止遮罩关闭
  document.addEventListener('click', preventClose, { once: true, capture: true })
}
function preventClose(e: MouseEvent) { e.stopPropagation() }
function onWheel(e: WheelEvent) {
  zoom.value = Math.max(0.5, Math.min(2, zoom.value + (e.deltaY > 0 ? -0.1 : 0.1)))
}
function onExpandedWheel(e: WheelEvent) {
  e.preventDefault()
  expandedZoom.value = Math.max(0.5, Math.min(2, expandedZoom.value + (e.deltaY > 0 ? -0.1 : 0.1)))
}

const expandedSceneStyle = computed(() => ({
  transform: `rotateX(${rotY.value}deg) rotateY(${rotX.value}deg) rotateZ(${rotZ.value}deg) scale(${expandedZoom.value})`,
}))

const sceneStyle = computed(() => ({
  transform: `rotateX(${rotY.value}deg) rotateY(${rotX.value}deg) rotateZ(${rotZ.value}deg) scale(${zoom.value})`,
}))

function ringStyle(i: number) {
  const s = 45 + i * 30
  const n = props.shells.length
  // 交替大角度：奇数层接近水平(10°)，偶数层接近垂直(85°)
  const tilt = i % 2 === 0 ? 10 + i * 5 : 85 - (Math.floor(i / 2) * 5)
  return {
    width: s+'px', height: s+'px',
    marginTop: -(s/2)+'px', marginLeft: -(s/2)+'px',
    transform: `rotateX(${tilt}deg) rotateY(${i * 35}deg)`,
  }
}

function spinnerStyle(i: number) {
  const speeds = [10, 13, 8, 15, 11, 9, 14]
  return {
    animation: `spin ${speeds[i] || 12}s linear infinite`,
    animationDirection: i % 2 === 0 ? 'normal' : 'reverse',
  }
}

function electronStyle(i: number, count: number, j: number) {
  const r = (45 + i * 30) / 2
  const angle = (360 / count) * j + i * 20
  const rad = angle * Math.PI / 180
  const x = Math.cos(rad) * r, y = Math.sin(rad) * r
  return { transform: `translate(${x}px, ${y}px)` }
}
</script>

<style scoped>
.atom-outer { width: 100%; height: 100%; display: flex; align-items: center; justify-content: center; position: relative; overflow: hidden; background: radial-gradient(circle, rgba(0,243,255,.03), transparent 70%); }
.atom-wrapper {
  position: absolute; inset: 0;
  cursor: grab; user-select: none; touch-action: none;
  display: flex; align-items: center; justify-content: center;
  perspective: 800px;
}
.atom-wrapper {
  position: absolute; inset: 0;
  cursor: grab; user-select: none; touch-action: none;
  display: flex; align-items: center; justify-content: center;
  background: radial-gradient(circle, rgba(0,243,255,.04), transparent 70%);
  perspective: 800px;
}
.atom-wrapper:active { cursor: grabbing; }
.atom-scene { position: relative; transform-style: preserve-3d; transition: transform .08s linear; }
.nucleus {
  position: absolute; top: 50%; left: 50%;
  width: 12px; height: 12px; margin: -6px 0 0 -6px;
  border-radius: 50%; background: #fff; z-index: 20;
  box-shadow: 0 0 15px #fff, 0 0 30px rgba(0,243,255,.6);
}
.nucleus-ring {
  position: absolute; top: 50%; left: 50%;
  width: 22px; height: 22px; margin: -11px 0 0 -11px;
  border-radius: 50%;
  border: 1px solid rgba(255,255,255,.2);
  transform: rotateX(85deg);
  z-index: 19;
}
.orbit-ring {
  position: absolute; top: 50%; left: 50%;
  border-radius: 50%;
  border: 1px solid rgba(255,255,255,.1);
  transform-style: preserve-3d; overflow: visible;
}
.orbit-spinner { width: 100%; height: 100%; position: relative; }
.electron { position: absolute; top: 50%; left: 50%; border-radius: 50%; }
.electron.inner { width: 7px; height: 7px; background: #00ccff; box-shadow: 0 0 6px #00ccff; margin: -3.5px 0 0 -3.5px; }
.electron.valence { width: 9px; height: 9px; background: #ff2255; box-shadow: 0 0 6px rgba(255,34,85,.5); margin: -4.5px 0 0 -4.5px; }
.atom-hint { position: absolute; bottom: 18px; color: rgba(255,255,255,.35); font-size: 16px; pointer-events: none; }
.expand-btn {
  position: absolute; top: 12px; left: 12px; z-index: 5;
  width: 40px; height: 40px; padding: 0;
  background: rgba(0,243,255,.1); border: 1px solid rgba(0,243,255,.3);
  border-radius: 6px; color: #00f3ff; cursor: pointer;
  display: flex; align-items: center; justify-content: center; transition: .2s; font-size: 20px;
}
.expand-btn:hover { background: rgba(0,243,255,.2); transform: scale(1.1); }
.expanded-overlay {
  position: fixed; inset: 0; z-index: 3000;
  background: rgba(0,0,0,.95); backdrop-filter: blur(15px);
  display: flex; align-items: center; justify-content: center;
}
.expanded-card {
  width: 90vw; height: 90vh; max-width: 960px; max-height: 540px;
  cursor: grab; user-select: none; touch-action: none;
  background: radial-gradient(ellipse at center, rgba(0,243,255,.04), transparent 60%), linear-gradient(135deg, #0a0a15, #151520);
  border: 1px solid rgba(0,243,255,.3); border-radius: 20px;
  box-shadow: 0 0 80px rgba(0,243,255,.15);
  display: flex; flex-direction: column; position: relative; perspective: 1200px; overflow: hidden;
}
.expanded-scene { position: relative; transform-style: preserve-3d; transition: transform .08s linear; flex: 1; display: flex; align-items: center; justify-content: center; }
.expanded-close {
  position: absolute; top: 20px; right: 20px; z-index: 10;
  font-size: 28px; color: #fff; cursor: pointer;
  width: 40px; height: 40px; background: rgba(0,0,0,.5); border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  border: 1px solid rgba(255,255,255,.1); padding: 0; line-height: 1;
}
.expanded-close:hover { background: rgba(255,0,0,.5); }
.expanded-header {
  display: flex; align-items: baseline; gap: 12px; padding: 12px 30px 0 20px;
}
.expanded-symbol { font-size: 42px; font-weight: 700; font-family: Arial; color: #00f3ff; }
.expanded-name { font-size: 24px; color: #fff; font-weight: 700; }
.expanded-en { font-size: 16px; color: #888; }
.expanded-shell-item { font-size: 13px; color: #aaa; display: flex; align-items: center; gap: 6px; padding: 4px 12px; background: rgba(255,255,255,.03); border-radius: 4px; white-space: nowrap; }
.expanded-shell-item.valence { color: #f69; background: rgba(255,0,85,.08); border: 1px solid rgba(255,0,85,.2); }
.shell-dot { width: 10px; height: 10px; border-radius: 50%; }
.shell-dot.inner { background: #00ccff; box-shadow: 0 0 4px #00ccff; }
.shell-dot.valence { background: #ff2255; box-shadow: 0 0 4px #ff2255; }
.expanded-hint { color: rgba(255,255,255,.3); font-size: 13px; pointer-events: none; padding: 6px 20px 0; text-align: center; }
.expanded-legend { padding: 6px 20px 12px 40px; display: flex; flex-wrap: wrap; justify-content: flex-start; gap: 16px; border-top: 1px solid rgba(255,255,255,.08); }
</style>

<style>
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
</style>
