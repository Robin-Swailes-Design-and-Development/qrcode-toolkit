<script setup lang="ts">
const props = defineProps<{
  max: number
  min: number
  step: number
  unit?: string
  default?: number
}>()

const value = defineModel<number>('modelValue', {
  type: Number,
})
</script>

<template>
  <div class="relative h-22px w-60 flex-auto">
    <input
      v-model.number="value"
      type="range"
      class="slider absolute bottom-0 left-0 right-0 top-0 z-10 w-full align-top"
      v-bind="props"
    >
    <span
      v-if="props.default != null"
      class="border-r border-base absolute bottom-0 top-0 h-full w-1px opacity-75"
      :style="{ left: `${(props.default - min) / (max - min) * 100}%` }"
    />
  </div>
  <div class="relative h-22px">
    <input
      v-model.number="value"
      type="number"
      class="border border-base rounded m-0 w-20 bg-secondary pl-2 align-top text-sm"
      v-bind="props"
    >
    <span
      v-if="props.unit"
      class="pointer-events-none absolute right-1 top-0.5 text-xs opacity-25"
    >
      {{ props.unit }}
    </span>
  </div>
</template>

<style>
.slider {
  appearance: none;
  height: 22px;
  outline: none;
  opacity: 0.7;
  -webkit-transition: .2s;
  transition: opacity .2s;
  @apply border border-base rounded overflow-hidden bg-secondary;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 5px;
  height: 22px;
  background: #3e5f78;
  cursor: pointer;
  z-index: 10;
}

.slider::-moz-range-thumb {
  width: 5px;
  height: 22px;
  background: #3e5f78;
  cursor: pointer;
  z-index: 10;
}
</style>