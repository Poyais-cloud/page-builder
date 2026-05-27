<template>
  <div
    class="text-component"
    :style="{
      fontSize: `${component.style.fontSize || 14}px`,
      fontWeight: component.style.fontWeight || 400,
      lineHeight: String(component.style.lineHeight || 1.5),
      color: component.style.color || '#3f4746',
      backgroundColor: component.style.backgroundColor || 'transparent',
      border: component.style.borderWidth ? `${component.style.borderWidth}px solid ${component.style.borderColor || '#d5cec2'}` : 'none',
      borderRadius: component.style.borderRadius ? `${component.style.borderRadius}px` : '0',
      padding: '2px 4px',
      boxSizing: 'border-box',
      whiteSpace: 'pre-wrap',
      wordBreak: 'break-word',
      textAlign: component.style.textAlign || 'left',
      opacity: component.style.opacity ?? 1
    }"
  >
    {{ textProps.content || '文本内容' }}
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { ComponentData, TextProps } from '@/types'

const props = defineProps<{
  component: ComponentData
}>()

const textProps = computed(() => props.component.props as TextProps)
</script>

<style scoped>
.text-component {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  min-height: 20px;
  letter-spacing: 0;
}
</style>
