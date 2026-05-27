<template>
  <input
    class="input-component"
    :type="inputProps.inputType || 'text'"
    :placeholder="inputProps.placeholder || '请输入内容'"
    :value="inputProps.value || ''"
    :style="{
      fontSize: `${component.style.fontSize || 14}px`,
      color: component.style.color || '#3f4746',
      backgroundColor: component.style.backgroundColor || '#faf8f3',
      border: component.style.borderWidth ? `${component.style.borderWidth}px solid ${component.style.borderColor || '#d5cec2'}` : '1px solid #d5cec2',
      borderRadius: component.style.borderRadius ? `${component.style.borderRadius}px` : '4px',
      padding: '8px 12px',
      boxSizing: 'border-box',
      width: '100%',
      height: '100%',
      opacity: component.style.opacity ?? 1
    }"
    disabled
  />
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { ComponentData, InputProps } from '@/types'

const props = defineProps<{
  component: ComponentData
}>()

const inputProps = computed(() => props.component.props as InputProps)
</script>

<style scoped>
.input-component {
  outline: none;
  transition: border-color 0.2s;
  min-height: 32px;
  box-shadow: 0 1px 2px rgba(73, 70, 61, 0.06);
}

.input-component::placeholder {
  color: #9a9489;
}
</style>
