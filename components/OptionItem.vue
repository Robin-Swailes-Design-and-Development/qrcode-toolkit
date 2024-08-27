<script setup lang="ts">
// import VTooltip from 'v-tooltip'
defineProps<{
  title: string
  nested?: boolean | number
  div?: boolean
  description?: string
}>()

const emit = defineEmits<{
  (event: 'reset'): void
}>()

function reset() {
  emit('reset')
}
</script>

<template>
  <component
    :is="div ? 'div' : 'label'"
    class="d-flex flex-row gap-2 align-items-center user-select-none"
  >
    <div class="w-35 d-flex align-items-center gap-1">
      <div
        v-if="nested"
        class="opacity-40"
        :class="typeof nested === 'number' ? 'bi bi-arrow-return-right' : ''"
        :style="typeof nested === 'number' ? { marginLeft: `${nested * 0.5 + 0.5}rem` } : { marginLeft: '0.25rem' }"
      ></div>
      <div
        v-if="!description"
        class="small opacity-75"
        @dblclick="reset"
      >
        {{ title }}
      </div>
      <div
        v-else
        class="small opacity-75"
        @dblclick="reset"
        data-bs-toggle="tooltip"
        data-bs-placement="left"
        :title="description"
      >
        {{ title }}
      </div>
    </div>
    <slot></slot>
  </component>
</template>
