<template>
  <div class="component-panel">
    <div class="panel-header">
      <div>
        <div class="panel-kicker">Component Library</div>
        <h3>组件资产库</h3>
        <p>拖拽到画布快速搭建营销页面</p>
      </div>
      <div class="panel-count">{{ componentList.length }}</div>
    </div>

    <div class="component-list">
      <div class="section-title">基础组件</div>
      <div
        v-for="component in baseComponents"
        :key="component.type"
        class="component-item"
        draggable="true"
        @dragstart="handleDragStart(component.type, $event)"
      >
        <div class="component-icon">
          <el-icon>
            <component :is="component.icon" />
          </el-icon>
        </div>
        <div class="component-meta">
          <div class="component-head">
            <div class="component-name">{{ component.label }}</div>
            <span class="component-tag">{{ component.category }}</span>
          </div>
          <div class="component-desc">{{ component.description }}</div>
        </div>
      </div>

      <div class="section-title marketing">营销场景</div>
      <div
        v-for="component in marketingComponents"
        :key="component.type"
        class="component-item marketing-item"
        draggable="true"
        @dragstart="handleDragStart(component.type, $event)"
      >
        <div class="component-icon marketing-icon">
          <el-icon>
            <component :is="component.icon" />
          </el-icon>
        </div>
        <div class="component-meta">
          <div class="component-head">
            <div class="component-name">{{ component.label }}</div>
            <span class="component-tag">{{ component.category }}</span>
          </div>
          <div class="component-desc">{{ component.description }}</div>
        </div>
      </div>

      <div class="section-title marketing">图层管理</div>
      <div v-if="sortedLayers.length" class="layer-list">
        <div
          v-for="layer in sortedLayers"
          :key="layer.id"
          class="layer-item"
          :class="{ active: currentComponent?.id === layer.id }"
          @click="selectLayer(layer.id)"
        >
          <span class="layer-index">{{ layer.style.zIndex }}</span>
          <span class="layer-name">{{ layer.name }}</span>
          <span class="layer-type">{{ layer.type }}</span>
        </div>
      </div>
      <div v-else class="empty-layer">当前暂无图层</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { componentProtocols } from './components/registry'
import { useEditorStore } from '@/stores/editor'
import { ComponentType } from '@/types'
import {
  CircleCheck,
  Document,
  EditPen,
  Picture,
  Tickets
} from '@element-plus/icons-vue'

const editorStore = useEditorStore()
const currentComponent = computed(() => editorStore.currentComponent)
const currentPage = computed(() => editorStore.currentPage)

const iconMap = {
  [ComponentType.TEXT]: Document,
  [ComponentType.IMAGE]: Picture,
  [ComponentType.BUTTON]: CircleCheck,
  [ComponentType.INPUT]: EditPen,
  [ComponentType.FORM]: Tickets
}

const componentList = computed(() =>
  componentProtocols.map((item) => ({
    ...item,
    icon: iconMap[item.type]
  }))
)

const baseComponents = computed(() => componentList.value.filter((item: (typeof componentList.value)[number]) => item.category === '基础'))
const marketingComponents = computed(() => componentList.value.filter((item: (typeof componentList.value)[number]) => item.category === '营销'))
const sortedLayers = computed(() => [...(currentPage.value?.components || [])].sort((a, b) => b.style.zIndex - a.style.zIndex))

const handleDragStart = (componentType: ComponentType, event: DragEvent) => {
  if (!event.dataTransfer) return
  event.dataTransfer.setData('componentType', componentType)
  event.dataTransfer.effectAllowed = 'copy'
}

const selectLayer = (componentId: string) => {
  const component = currentPage.value?.components.find((item) => item.id === componentId) || null
  editorStore.selectComponent(component)
}
</script>

<style scoped>
.component-panel {
  width: 300px;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  display: flex;
  flex-direction: column;
  box-shadow: inset -1px 0 0 rgba(73, 70, 61, 0.03);
}

.panel-header {
  padding: 18px 16px 16px;
  border-bottom: 1px solid var(--color-border-soft);
  background: linear-gradient(180deg, #faf8f3 0%, #f4f1ea 100%);
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
}

.panel-kicker {
  margin-bottom: 4px;
  color: var(--color-primary);
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 0.04em;
  text-transform: uppercase;
}

.panel-header h3 {
  margin: 0;
  font-size: 16px;
  font-weight: 800;
  color: var(--color-heading);
}

.panel-header p {
  margin: 6px 0 0;
  font-size: 12px;
  color: var(--color-text-muted);
}

.panel-count {
  min-width: 34px;
  height: 24px;
  padding: 0 8px;
  border-radius: 999px;
  background: var(--color-primary-soft);
  color: var(--color-primary);
  font-size: 12px;
  font-weight: 800;
  display: grid;
  place-items: center;
}

.component-list {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
}

.section-title {
  margin: 4px 0 12px;
  color: var(--color-text-muted);
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}

.section-title.marketing {
  margin-top: 18px;
}

.component-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 14px;
  margin-bottom: 12px;
  border: 1px solid var(--color-border-soft);
  border-radius: var(--radius-md);
  background: var(--color-surface);
  cursor: grab;
  transition: 0.2s ease;
  user-select: none;
}

.component-item:hover {
  transform: translateY(-1px);
  border-color: rgba(111, 133, 131, 0.42);
  box-shadow: var(--shadow-soft);
}

.marketing-item {
  background: #fbfaf6;
}

.component-icon {
  width: 44px;
  height: 44px;
  border-radius: var(--radius-md);
  background: var(--color-primary-soft);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.marketing-icon {
  background: var(--color-accent-soft);
}

.component-icon .el-icon {
  font-size: 20px;
  color: var(--color-primary);
}

.component-meta {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 0;
  flex: 1;
}

.component-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.component-name {
  font-size: 14px;
  font-weight: 700;
  color: var(--color-heading);
}

.component-desc {
  font-size: 12px;
  line-height: 1.5;
  color: var(--color-text-muted);
}

.component-tag {
  flex-shrink: 0;
  padding: 2px 8px;
  border-radius: 999px;
  background: var(--color-surface-soft);
  color: var(--color-text-muted);
  font-size: 11px;
  font-weight: 700;
}

.layer-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.layer-item {
  display: grid;
  grid-template-columns: 28px 1fr auto;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  border: 1px solid var(--color-border-soft);
  border-radius: var(--radius-md);
  background: var(--color-surface);
  cursor: pointer;
  transition: 0.16s ease;
}

.layer-item.active {
  border-color: rgba(111, 133, 131, 0.44);
  background: var(--color-primary-soft);
  box-shadow: inset 2px 0 0 var(--color-primary);
}

.layer-index {
  width: 24px;
  height: 24px;
  line-height: 24px;
  text-align: center;
  border-radius: 999px;
  background: var(--color-surface-soft);
  color: var(--color-text);
  font-size: 12px;
  font-weight: 700;
}

.layer-name {
  font-size: 13px;
  color: var(--color-heading);
  font-weight: 700;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.layer-type,
.empty-layer {
  font-size: 12px;
  color: var(--color-text-muted);
}

.empty-layer {
  padding: 14px 10px;
  border: 1px dashed var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface-soft);
  text-align: center;
}
</style>
