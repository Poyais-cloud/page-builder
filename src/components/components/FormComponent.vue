<template>
  <div
    class="form-component"
    :style="containerStyle"
  >
    <div class="form-title">{{ formProps.title || '活动报名表单' }}</div>

    <div class="form-fields">
      <label
        v-for="field in formProps.fields"
        :key="field.id"
        class="form-field"
      >
        <span>{{ field.label }}<em v-if="field.required">*</em></span>
        <input
          :type="field.type"
          :placeholder="field.placeholder"
          disabled
        />
      </label>
    </div>

    <button class="submit-button" type="button">
      {{ formProps.submitText || '立即提交' }}
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { CSSProperties } from 'vue'
import type { ComponentData, FormProps } from '@/types'

const props = defineProps<{
  component: ComponentData
}>()

const formProps = computed(() => props.component.props as FormProps)

const containerStyle = computed<CSSProperties>(() => ({
  backgroundColor: props.component.style.backgroundColor || '#faf8f3',
  border: props.component.style.borderWidth
    ? `${props.component.style.borderWidth}px solid ${props.component.style.borderColor || '#d5cec2'}`
    : '1px solid #d5cec2',
  borderRadius: `${props.component.style.borderRadius ?? 12}px`,
  boxSizing: 'border-box',
  padding: '20px',
  width: '100%',
  height: '100%',
  opacity: props.component.style.opacity ?? 1
}))
</script>

<style scoped>
.form-component {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
  height: 100%;
  box-shadow: 0 18px 40px rgba(73, 70, 61, 0.1);
  overflow: auto;
}

.form-title {
  font-size: 18px;
  font-weight: 700;
  color: #3f4746;
  line-height: 1.4;
}

.form-fields {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 12px;
  min-height: 0;
}

.form-field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 13px;
  color: #596261;
  font-weight: 700;
}

.form-field em {
  color: #9b7467;
  font-style: normal;
  margin-left: 2px;
}

.form-field input {
  width: 100%;
  height: 38px;
  border: 1px solid #d5cec2;
  border-radius: 8px;
  padding: 0 12px;
  box-sizing: border-box;
  background: #f4f1ea;
  color: #596261;
}

.submit-button {
  height: 40px;
  min-height: 40px;
  border: none;
  border-radius: 999px;
  background: #6f8583;
  color: #faf8f3;
  font-weight: 600;
  box-shadow: 0 12px 24px rgba(82, 107, 105, 0.24);
}
</style>
