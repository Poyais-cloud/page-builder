<template>
  <div class="page-editor-container">
    <div class="editor-header">
      <div class="header-left">
        <div class="brand-mark">
          <el-icon><EditPen /></el-icon>
        </div>
        <div>
          <div class="header-kicker">Yuan Drag Builder</div>
          <h1>营销页面搭建平台</h1>
          <div class="header-subtitle">
            <span>{{ currentPage?.meta.title || '未命名页面' }}</span>
            <span>场景：{{ currentPage?.meta.scene || 'marketing' }}</span>
            <span>组件数：{{ currentPage?.components.length || 0 }}</span>
            <span v-if="lastSavedAt">最近保存：{{ lastSavedAt }}</span>
          </div>
        </div>
      </div>

      <div class="header-center">
        <el-button-group class="history-actions">
          <el-button @click="historyStore.undo()" :disabled="!historyStore.canUndo">
            <el-icon><RefreshLeft /></el-icon>
            撤销
          </el-button>
          <el-button @click="historyStore.redo()" :disabled="!historyStore.canRedo">
            <el-icon><RefreshRight /></el-icon>
            重做
          </el-button>
        </el-button-group>

        <el-button @click="previewVisible = true" type="primary" plain>
          <el-icon><View /></el-icon>
          预览
        </el-button>
        <el-button @click="handleExport" type="success" plain>
          <el-icon><Download /></el-icon>
          导出 JSON
        </el-button>
      </div>

      <div class="header-right">
        <el-upload :show-file-list="false" accept="application/json" :auto-upload="false" @change="handleImport">
          <el-button>
            <el-icon><Upload /></el-icon>
            导入 JSON
          </el-button>
        </el-upload>
        <el-button @click="handleSave">
          <el-icon><Document /></el-icon>
          保存
        </el-button>
        <el-button @click="handleNewPage">
          <el-icon><Plus /></el-icon>
          新建页面
        </el-button>
      </div>
    </div>

    <div class="editor-body">
      <ComponentPanel />
      <EditorCanvas />
      <PropertyPanel />
    </div>

    <el-dialog v-model="previewVisible" title="页面预览" width="90%" top="4vh">
      <div class="preview-shell">
        <div class="preview-device-bar">实时渲染预览</div>
        <div class="preview-content" :style="previewStyle">
          <div v-for="component in sortedComponents" :key="component.id" class="preview-component" :style="previewComponentStyle(component)">
            <component :is="getRenderer(component.type)" :component="component" />
          </div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { ElMessage } from 'element-plus'
import type { UploadFile } from 'element-plus'
import { useEditorStore } from '@/stores/editor'
import { useHistoryStore } from '@/stores/history'
import type { ComponentData, ComponentType } from '@/types'
import ComponentPanel from './ComponentPanel.vue'
import EditorCanvas from './EditorCanvas.vue'
import PropertyPanel from './PropertyPanel.vue'
import { componentRendererMap } from './components/registry'
import { Document, Download, EditPen, Plus, RefreshLeft, RefreshRight, Upload, View } from '@element-plus/icons-vue'

const editorStore = useEditorStore()
const historyStore = useHistoryStore()
const previewVisible = ref(false)

const currentPage = computed(() => editorStore.currentPage)
const lastSavedAt = computed(() => editorStore.lastSavedAt)
const sortedComponents = computed(() => [...(currentPage.value?.components || [])].sort((a, b) => a.style.zIndex - b.style.zIndex))
const getRenderer = (type: ComponentType) => componentRendererMap[type]

const previewStyle = computed(() => ({
  width: `${currentPage.value?.style.width || 1200}px`,
  height: `${currentPage.value?.style.height || 800}px`,
  backgroundColor: currentPage.value?.style.backgroundColor || '#faf8f3',
  backgroundImage: currentPage.value?.style.backgroundImage || 'none'
}))

const previewComponentStyle = (component: ComponentData) => ({
  top: `${component.style.top}px`,
  left: `${component.style.left}px`,
  width: `${component.style.width}px`,
  height: `${component.style.height}px`,
  zIndex: component.style.zIndex,
  transform: `rotate(${component.style.rotate}deg)`
})

const handleExport = () => {
  const data = editorStore.exportPageData()
  if (!data) {
    ElMessage.warning('当前没有可导出的页面数据')
    return
  }

  const blob = new Blob([data], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `${currentPage.value?.meta.title || 'marketing-page'}.json`
  link.click()
  URL.revokeObjectURL(url)
}

const handleSave = () => {
  editorStore.persistPage()
  ElMessage.success('页面已持久化保存')
}

const handleNewPage = () => {
  editorStore.createNewPage('新的营销活动页')
  historyStore.clearHistory()
  ElMessage.success('已创建新页面')
}

const handleImport = async (file: UploadFile) => {
  const raw = file.raw
  if (!raw) return

  try {
    const text = await raw.text()
    editorStore.importPageData(text)
    historyStore.clearHistory()
    editorStore.persistPage()
    ElMessage.success('页面 JSON 导入成功')
  } catch {
    ElMessage.error('导入失败，请检查 JSON 格式')
  }
}
</script>

<style scoped>
.page-editor-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: var(--color-background);
  overflow: hidden;
}

.editor-header {
  height: 76px;
  padding: 0 18px;
  background: rgba(250, 248, 243, 0.96);
  border-bottom: 1px solid var(--color-border);
  display: grid;
  grid-template-columns: minmax(360px, 1fr) auto minmax(300px, 1fr);
  align-items: center;
  gap: 18px;
  box-shadow: var(--shadow-panel);
  z-index: 5;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 0;
}

.brand-mark {
  width: 42px;
  height: 42px;
  border-radius: var(--radius-md);
  color: #faf8f3;
  background: linear-gradient(135deg, var(--color-primary-strong), #7f929b);
  display: grid;
  place-items: center;
  box-shadow: 0 10px 22px rgba(82, 107, 105, 0.24);
}

.brand-mark .el-icon {
  font-size: 20px;
}

.header-kicker {
  margin-bottom: 1px;
  color: var(--color-primary);
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 0.04em;
  text-transform: uppercase;
}

.header-left h1 {
  margin: 0;
  color: var(--color-heading);
  font-size: 19px;
  font-weight: 800;
  line-height: 1.2;
}

.header-subtitle {
  margin-top: 7px;
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  color: var(--color-text-muted);
  font-size: 12px;
}

.header-subtitle span {
  max-width: 180px;
  padding: 2px 8px;
  border: 1px solid var(--color-border-soft);
  border-radius: 999px;
  background: var(--color-surface-soft);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.header-center,
.header-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.header-center {
  justify-content: center;
}

.header-right {
  justify-content: flex-end;
}

.history-actions {
  margin-right: 4px;
}

.editor-body {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.preview-shell {
  height: calc(92vh - 88px);
  padding: 20px;
  background: var(--color-background);
  overflow: auto;
}

.preview-device-bar {
  width: min(100%, 1200px);
  margin: 0 auto 12px;
  color: var(--color-text);
  font-size: 13px;
  font-weight: 800;
}

.preview-content {
  position: relative;
  margin: 0 auto 20px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  overflow: hidden;
  box-shadow: var(--shadow-soft);
}

.preview-component {
  position: absolute;
}

.preview-component > * {
  width: 100%;
  height: 100%;
}
</style>
