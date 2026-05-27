<template>
  <div class="editor-canvas">
    <div class="canvas-toolbar">
      <div class="toolbar-left">
        <el-button-group class="zoom-controls">
          <el-button @click="zoomOut" :disabled="canvasScale <= 0.3"><el-icon><ZoomOut /></el-icon></el-button>
          <el-button class="scale-readout">{{ Math.round(canvasScale * 100) }}%</el-button>
          <el-button @click="zoomIn" :disabled="canvasScale >= 2"><el-icon><ZoomIn /></el-icon></el-button>
          <el-button @click="fitToViewport"><el-icon><FullScreen /></el-icon></el-button>
        </el-button-group>
        <div class="toolbar-toggles">
          <el-switch v-model="snapToGrid" active-text="网格吸附" />
          <el-switch v-model="showGuidelines" active-text="对齐线" />
        </div>
      </div>
      <div class="canvas-metrics">
        <span>{{ pageWidth }} x {{ pageHeight }}</span>
        <span>{{ componentCount }} layers</span>
      </div>
    </div>

    <div
      ref="viewportRef"
      class="canvas-viewport"
      tabindex="0"
      @keydown="handleKeydown"
      @mousedown.self="clearSelection"
      @dragover.prevent
      @drop="handleDrop"
    >
      <div class="canvas-stage" :style="stageStyle" @mousedown.self="clearSelection">
        <div
          ref="backgroundRef"
          class="canvas-background"
          :style="canvasBgStyle"
          @mousedown.self="clearSelection"
        >
          <div class="canvas-dots"></div>
          <div
            v-for="component in sortedComponents"
            :key="component.id"
            class="component-wrapper"
            :class="{ selected: currentComponent?.id === component.id }"
            :style="wrapperStyle(component)"
            @mousedown.stop="startDrag(component, $event)"
          >
            <component :is="getRenderer(component.type)" :component="component" class="component-content" />

            <div
              class="component-hitbox"
              :class="{ active: currentComponent?.id === component.id }"
              @click.stop="editorStore.selectComponent(component)"
            >
              <div v-if="currentComponent?.id === component.id" class="selection-box">
                <div class="selection-label">{{ component.name }}</div>
                <div class="rotate-badge">{{ component.style.rotate }}°</div>
                <div class="resize-handles">
                  <div class="handle handle-tl" @mousedown.stop.prevent="beginResize(component, 'tl', $event)"></div>
                  <div class="handle handle-tr" @mousedown.stop.prevent="beginResize(component, 'tr', $event)"></div>
                  <div class="handle handle-bl" @mousedown.stop.prevent="beginResize(component, 'bl', $event)"></div>
                  <div class="handle handle-br" @mousedown.stop.prevent="beginResize(component, 'br', $event)"></div>
                  <div class="handle handle-t" @mousedown.stop.prevent="beginResize(component, 't', $event)"></div>
                  <div class="handle handle-r" @mousedown.stop.prevent="beginResize(component, 'r', $event)"></div>
                  <div class="handle handle-b" @mousedown.stop.prevent="beginResize(component, 'b', $event)"></div>
                  <div class="handle handle-l" @mousedown.stop.prevent="beginResize(component, 'l', $event)"></div>
                </div>
              </div>
            </div>
          </div>
          <div v-if="showGuidelines && guides.x !== null" class="guide guide-x" :style="{ left: `${guides.x}px` }"></div>
          <div v-if="showGuidelines && guides.y !== null" class="guide guide-y" :style="{ top: `${guides.y}px` }"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref, toRaw, watch } from 'vue'
import { useEditorStore } from '@/stores/editor'
import type { ComponentData } from '@/types'
import { ComponentType } from '@/types'
import { componentRendererMap } from './components/registry'

const editorStore = useEditorStore()
const viewportRef = ref<HTMLElement | null>(null)
const backgroundRef = ref<HTMLElement | null>(null)
const guides = ref<{ x: number | null; y: number | null }>({ x: null, y: null })

const currentPage = computed(() => editorStore.currentPage)
const currentComponent = computed(() => editorStore.currentComponent)
const canvasScale = computed(() => editorStore.canvasScale)
const snapToGrid = computed({ get: () => editorStore.snapToGrid, set: (v) => editorStore.setSnapToGrid(v) })
const showGuidelines = computed({ get: () => editorStore.showGuidelines, set: (v) => editorStore.setShowGuidelines(v) })
const pageWidth = computed(() => currentPage.value?.style.width || 1200)
const pageHeight = computed(() => currentPage.value?.style.height || 800)
const sortedComponents = computed(() => [...(currentPage.value?.components || [])].sort((a, b) => a.style.zIndex - b.style.zIndex))
const componentCount = computed(() => currentPage.value?.components.length || 0)

// stage 用缩放后的视觉尺寸占位
const stageStyle = computed(() => ({
  width: `${pageWidth.value * canvasScale.value + 120}px`,
  minHeight: `${pageHeight.value * canvasScale.value + 120}px`
}))

// canvas 用逻辑尺寸，再通过 scale 缩放
const canvasBgStyle = computed(() => ({
  width: `${pageWidth.value}px`,
  height: `${pageHeight.value}px`,
  backgroundColor: currentPage.value?.style.backgroundColor || '#faf8f3',
  backgroundImage: currentPage.value?.style.backgroundImage || 'none',
  transform: `scale(${canvasScale.value})`,
  transformOrigin: 'top center'
}))

const getRenderer = (type: ComponentType) => componentRendererMap[type] || componentRendererMap[ComponentType.TEXT]
const wrapperStyle = (component: ComponentData) => ({
  top: `${component.style.top}px`,
  left: `${component.style.left}px`,
  width: `${component.style.width}px`,
  height: `${component.style.height}px`,
  zIndex: component.style.zIndex,
  transform: `rotate(${component.style.rotate}deg)`
})
const align = (value: number) => snapToGrid.value ? Math.round(value / editorStore.GRID_SIZE) * editorStore.GRID_SIZE : value
const clearGuides = () => { guides.value = { x: null, y: null } }
const updateGuides = (left: number, top: number, width: number, height: number) => {
  const near = (targets: number[], val: number) => targets.find((n) => Math.abs(n - val) <= 4) ?? null
  guides.value = {
    x: near([0, pageWidth.value / 2, pageWidth.value], left) ?? near([0, pageWidth.value / 2, pageWidth.value], left + width / 2),
    y: near([0, pageHeight.value / 2, pageHeight.value], top) ?? near([0, pageHeight.value / 2, pageHeight.value], top + height / 2)
  }
}

const fitToViewport = async () => {
  await nextTick()
  await nextTick()
  if (!viewportRef.value || !currentPage.value) return
  const availableWidth = viewportRef.value.clientWidth - 80
  const availableHeight = viewportRef.value.clientHeight - 80
  const widthScale = availableWidth / pageWidth.value
  const heightScale = availableHeight / pageHeight.value
  const nextScale = Math.max(0.3, Math.min(1, Math.min(widthScale, heightScale)))
  editorStore.setCanvasScale(Number(nextScale.toFixed(2)))
}

const zoomIn = () => editorStore.setCanvasScale(Math.min(canvasScale.value + 0.1, 2))
const zoomOut = () => editorStore.setCanvasScale(Math.max(canvasScale.value - 0.1, 0.3))
const clearSelection = () => editorStore.selectComponent(null)

const handleDrop = (event: DragEvent) => {
  event.preventDefault()
  if (!event.dataTransfer || !currentPage.value || !backgroundRef.value) return
  const componentType = event.dataTransfer.getData('componentType') as ComponentType
  if (!componentType) return
  // getBoundingClientRect 返回的是 scale 后的视觉坐标，要除以 scale 才是逻辑坐标
  const rect = backgroundRef.value.getBoundingClientRect()
  const left = Math.max(0, align((event.clientX - rect.left) / canvasScale.value))
  const top = Math.max(0, align((event.clientY - rect.top) / canvasScale.value))
  editorStore.addComponent(componentType, { style: { left, top } })
}

// 画布内拖拽
let dragId = ''
let dragStartX = 0
let dragStartY = 0
let dragInitial: ComponentData['style'] | null = null

const startDrag = (component: ComponentData, event: MouseEvent) => {
  if (event.button !== 0) return
  event.preventDefault()
  editorStore.selectComponent(component)
  dragId = component.id
  dragInitial = JSON.parse(JSON.stringify(toRaw(component.style)))
  dragStartX = event.clientX
  dragStartY = event.clientY
  document.body.style.userSelect = 'none'
  document.body.style.cursor = 'grabbing'
  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', stopDrag)
}

const onDrag = (event: MouseEvent) => {
  if (!dragId || !dragInitial) return
  const left = Math.max(0, align(dragInitial.left + (event.clientX - dragStartX) / canvasScale.value))
  const top = Math.max(0, align(dragInitial.top + (event.clientY - dragStartY) / canvasScale.value))
  editorStore.applyComponentStyle(dragId, { left, top })
  updateGuides(left, top, dragInitial.width, dragInitial.height)
}

const stopDrag = () => {
  if (dragId && currentComponent.value) {
    editorStore.commitComponentStyle(currentComponent.value.id, JSON.parse(JSON.stringify(toRaw(currentComponent.value.style))))
  }
  dragId = ''
  dragInitial = null
  clearGuides()
  document.body.style.userSelect = ''
  document.body.style.cursor = ''
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
}

// 缩放
let resizeId = ''
let resizeDir = ''
let resizeStartX = 0
let resizeStartY = 0
let resizeInitial: ComponentData['style'] | null = null

const beginResize = (component: ComponentData, direction: string, event: MouseEvent) => {
  if (event.button !== 0) return
  event.preventDefault()
  editorStore.selectComponent(component)
  resizeId = component.id
  resizeDir = direction
  resizeInitial = JSON.parse(JSON.stringify(toRaw(component.style)))
  resizeStartX = event.clientX
  resizeStartY = event.clientY
  document.body.style.userSelect = 'none'
  document.body.style.cursor = 'nwse-resize'
  document.addEventListener('mousemove', onResize)
  document.addEventListener('mouseup', stopResize)
}

const onResize = (event: MouseEvent) => {
  if (!resizeId || !resizeInitial) return
  const dx = (event.clientX - resizeStartX) / canvasScale.value
  const dy = (event.clientY - resizeStartY) / canvasScale.value
  let { left, top, width, height } = resizeInitial
  if (resizeDir.includes('l')) { width = Math.max(40, resizeInitial.width - dx); left = resizeInitial.left + (resizeInitial.width - width) }
  if (resizeDir.includes('r')) width = Math.max(40, resizeInitial.width + dx)
  if (resizeDir.includes('t')) { height = Math.max(40, resizeInitial.height - dy); top = resizeInitial.top + (resizeInitial.height - height) }
  if (resizeDir.includes('b')) height = Math.max(40, resizeInitial.height + dy)
  left = Math.max(0, align(left)); top = Math.max(0, align(top)); width = align(width); height = align(height)
  editorStore.applyComponentStyle(resizeId, { left, top, width, height })
  updateGuides(left, top, width, height)
}

const stopResize = () => {
  if (resizeId && currentComponent.value) {
    editorStore.commitComponentStyle(currentComponent.value.id, JSON.parse(JSON.stringify(toRaw(currentComponent.value.style))))
  }
  resizeId = ''
  resizeDir = ''
  resizeInitial = null
  clearGuides()
  document.body.style.userSelect = ''
  document.body.style.cursor = ''
  document.removeEventListener('mousemove', onResize)
  document.removeEventListener('mouseup', stopResize)
}

const handleKeydown = (event: KeyboardEvent) => {
  if (!currentComponent.value) return
  const step = event.shiftKey ? 10 : 1
  if (event.key === 'Delete' || event.key === 'Backspace') { editorStore.deleteComponent(currentComponent.value.id); return }
  if (event.key === 'ArrowUp') { event.preventDefault(); editorStore.nudgeComponent(currentComponent.value.id, 0, -step) }
  if (event.key === 'ArrowDown') { event.preventDefault(); editorStore.nudgeComponent(currentComponent.value.id, 0, step) }
  if (event.key === 'ArrowLeft') { event.preventDefault(); editorStore.nudgeComponent(currentComponent.value.id, -step, 0) }
  if (event.key === 'ArrowRight') { event.preventDefault(); editorStore.nudgeComponent(currentComponent.value.id, step, 0) }
}

onMounted(async () => {
  editorStore.loadPersistedPage()
  viewportRef.value?.focus()
  await fitToViewport()
  setTimeout(() => { fitToViewport() }, 0)
  window.addEventListener('resize', fitToViewport)
})

watch([pageWidth, pageHeight, componentCount], () => {
  fitToViewport()
})

onUnmounted(() => {
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
  document.removeEventListener('mousemove', onResize)
  document.removeEventListener('mouseup', stopResize)
  window.removeEventListener('resize', fitToViewport)
})
</script>

<style scoped>
.editor-canvas {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: var(--color-background);
  overflow: hidden;
}

.canvas-toolbar {
  height: 58px;
  padding: 0 18px;
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border-soft);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toolbar-left {
  display: flex;
  align-items: center;
  gap: 14px;
}

.zoom-controls {
  box-shadow: 0 0 0 1px var(--color-border-soft);
  border-radius: var(--radius-sm);
}

.scale-readout {
  min-width: 64px;
  color: var(--color-heading);
  font-weight: 800;
}

.toolbar-toggles {
  display: flex;
  align-items: center;
  gap: 14px;
  padding-left: 2px;
}

.canvas-metrics {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--color-text-muted);
  font-size: 12px;
}

.canvas-metrics span {
  padding: 3px 8px;
  border: 1px solid var(--color-border-soft);
  border-radius: 999px;
  background: var(--color-surface-soft);
  font-weight: 700;
}

.canvas-viewport {
  flex: 1;
  overflow: auto;
  padding: 26px;
  outline: none;
  background:
    linear-gradient(90deg, rgba(135, 123, 104, 0.16) 1px, transparent 1px),
    linear-gradient(180deg, rgba(135, 123, 104, 0.16) 1px, transparent 1px),
    var(--color-background);
  background-size: 24px 24px;
}

.canvas-stage {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 22px 0 52px;
  box-sizing: border-box;
}

.canvas-background {
  position: relative;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  box-shadow: 0 24px 60px rgba(73, 70, 61, 0.14);
  overflow: hidden;
}

.canvas-dots {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(rgba(111, 133, 131, 0.3) 0.8px, transparent 0.8px);
  background-size: 16px 16px;
  pointer-events: none;
  border-radius: var(--radius-md);
}

.component-wrapper {
  position: absolute;
  cursor: move;
  user-select: none;
}

.component-wrapper.selected {
  z-index: 999 !important;
}

.component-content {
  position: relative;
  z-index: 1;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: hidden;
}

.component-hitbox {
  position: absolute;
  inset: 0;
  z-index: 2;
  cursor: move;
  background: transparent;
  pointer-events: auto;
}

.component-hitbox.active {
  cursor: grab;
}

.selection-box {
  position: absolute;
  inset: -2px;
  border: 2px solid var(--color-primary);
  border-radius: var(--radius-md);
  /* box 本身不拦截事件，但 handle 子元素仍然可点 */
  pointer-events: none;
  box-shadow: 0 0 0 3px rgba(111, 133, 131, 0.18);
}

.selection-label,
.rotate-badge {
  position: absolute;
  top: -28px;
  height: 22px;
  line-height: 22px;
  padding: 0 8px;
  border-radius: var(--radius-sm);
  font-size: 12px;
  font-weight: 700;
  color: #faf8f3;
  background: var(--color-primary);
}

.selection-label {
  left: 0;
}

.rotate-badge {
  right: 0;
  background: var(--color-accent);
}

.resize-handles {
  position: absolute;
  inset: 0;
  /* 容器不拦截，子元素 pointer-events: all 仍生效 */
  pointer-events: none;
}
.handle {
  position: absolute;
  width: 10px;
  height: 10px;
  border: 2px solid var(--color-surface);
  border-radius: 50%;
  background: var(--color-primary);
  /* handle 自身恢复事件响应 */
  pointer-events: all;
  z-index: 10;
  box-shadow: 0 2px 8px rgba(111, 133, 131, 0.3);
}

.handle-tl {
  top: -6px;
  left: -6px;
  cursor: nwse-resize;
}

.handle-tr {
  top: -6px;
  right: -6px;
  cursor: nesw-resize;
}

.handle-bl {
  bottom: -6px;
  left: -6px;
  cursor: nesw-resize;
}

.handle-br {
  right: -6px;
  bottom: -6px;
  cursor: nwse-resize;
}

.handle-t {
  top: -6px;
  left: calc(50% - 5px);
  cursor: ns-resize;
}

.handle-r {
  top: calc(50% - 5px);
  right: -6px;
  cursor: ew-resize;
}

.handle-b {
  bottom: -6px;
  left: calc(50% - 5px);
  cursor: ns-resize;
}

.handle-l {
  top: calc(50% - 5px);
  left: -6px;
  cursor: ew-resize;
}

.guide {
  position: absolute;
  z-index: 998;
  pointer-events: none;
  background: rgba(155, 116, 103, 0.82);
}

.guide-x {
  top: 0;
  bottom: 0;
  width: 1px;
}

.guide-y {
  right: 0;
  left: 0;
  height: 1px;
}
</style>
