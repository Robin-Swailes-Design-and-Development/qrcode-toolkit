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
    class="flex flex-row gap-2 items-center select-none"
  >
    <div class="w-35 flex items-center gap-1">
      <div
        v-if="nested"
        class="opacity-40"
        :class="typeof nested === 'number' ? 'i-ri-corner-down-right-line' : ''"
        :style="typeof nested === 'number' ? { marginLeft: `${nested * 0.5 + 0.5}rem` } : { marginLeft: '0.25rem' }"
      />
      <div
        v-if="!description"
        class="text-sm opacity-75"
        @dblclick="reset"
      >
        {{ title }}
      </div>
      <VTooltip
        v-else
        placement="left"
        distance="10"
      >
        <div
          class="text-sm opacity-75"
          @dblclick="reset"
        >
          {{ title }}
        </div>
        <template #popper>
          <div class="text-sm">
            {{ description }}
          </div>
        </template>
      </VTooltip>
    </div>
    <slot />
  </component>
</template>