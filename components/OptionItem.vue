<script setup lang="ts">
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
    class="d-flex flex-row align-items-center user-select-none mb-3"
  >
    <div class="w-35 d-flex align-items-center">
      <div
        v-if="nested"
        class="text-muted me-1"
        :class="typeof nested === 'number' ? 'bi bi-arrow-return-right' : ''"
        :style="typeof nested === 'number' ? { marginLeft: `${nested * 0.5 + 0.5}rem` } : { marginLeft: '0.25rem' }"
      />
      <div
        v-if="!description"
        class="small text-muted"
        @dblclick="reset"
      >
        {{ title }}
      </div>
      <div
        v-else
        class="small text-muted"
        @dblclick="reset"
        data-bs-toggle="tooltip"
        data-bs-placement="left"
        :title="description"
      >
        {{ title }}
      </div>
    </div>
    <div class="ms-2">
      <slot />
    </div>
  </component>
</template>