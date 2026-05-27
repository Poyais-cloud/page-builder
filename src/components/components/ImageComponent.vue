<template>
  <div
    class="image-component"
    :style="{
      border: component.style.borderWidth ? `${component.style.borderWidth}px solid ${component.style.borderColor || '#d5cec2'}` : 'none',
      borderRadius: component.style.borderRadius ? `${component.style.borderRadius}px` : '0',
      overflow: 'hidden',
      backgroundColor: component.style.backgroundColor || '#e8e1d7',
      opacity: component.style.opacity ?? 1
    }"
  >
    <img
      v-if="imageProps.src"
      :src="imageProps.src"
      :alt="imageProps.alt || '图片'"
      class="image-content"
      :style="{
        width: '100%',
        height: '100%',
        objectFit: imageProps.objectFit || 'cover'
      }"
    />
    <div v-else class="image-placeholder">
      <el-icon><Picture /></el-icon>
      <span>请设置图片地址</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { Picture } from '@element-plus/icons-vue'
import type { ComponentData, ImageProps } from '@/types'

const props = defineProps<{
  component: ComponentData
}>()

const imageProps = computed(() => props.component.props as ImageProps)
</script>

<style scoped>
.image-component {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image:
    linear-gradient(135deg, rgba(111, 133, 131, 0.14), rgba(155, 116, 103, 0.1)),
    linear-gradient(90deg, rgba(135, 123, 104, 0.18) 1px, transparent 1px),
    linear-gradient(180deg, rgba(135, 123, 104, 0.18) 1px, transparent 1px);
  background-size: auto, 18px 18px, 18px 18px;
}

.image-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #7d8580;
  font-size: 12px;
  font-weight: 700;
}

.image-content {
  display: block;
}
</style>
