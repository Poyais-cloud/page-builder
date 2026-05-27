<template>
  <button
    class="button-component"
    :style="{
      fontSize: `${component.style.fontSize || 14}px`,
      fontWeight: component.style.fontWeight || 600,
      color: component.style.color || '#faf8f3',
      backgroundColor: component.style.backgroundColor || '#6f8583',
      border: component.style.borderWidth ? `${component.style.borderWidth}px solid ${component.style.borderColor || '#6f8583'}` : 'none',
      borderRadius: component.style.borderRadius ? `${component.style.borderRadius}px` : '4px',
      boxSizing: 'border-box',
      cursor: 'pointer',
      width: '100%',
      height: '100%',
      opacity: component.style.opacity ?? 1
    }"
    type="button"
  >
    {{ buttonProps.content || '按钮' }}
  </button>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { ButtonProps, ComponentData } from '@/types'

const props = defineProps<{
  component: ComponentData
}>()

const buttonProps = computed(() => props.component.props as ButtonProps)
</script>

<style scoped>
.button-component {
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  min-height: 32px;
  box-shadow: 0 12px 24px rgba(82, 107, 105, 0.24);
  letter-spacing: 0;
}

.button-component:hover {
  transform: translateY(-1px);
  opacity: 0.96;
}

.button-component:active {
  transform: scale(0.98);
}
</style>
