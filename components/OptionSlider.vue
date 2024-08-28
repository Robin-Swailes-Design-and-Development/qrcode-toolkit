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
  <div class="d-flex align-items-center">
    <div class="flex-grow-1 me-2 position-relative" style="height: 22px;">
      <input
        v-model.number="value"
        type="range"
        class="slider position-absolute bottom-0 start-0 end-0 top-0 w-100 align-top border border-secondary rounded overflow-hidden"
        v-bind="props"
      >
      <span
        v-if="props.default != null"
        class="border-end border-secondary position-absolute bottom-0 top-0 h-100"
        style="width: 1px; opacity: 0.75;"
        :style="{ left: `${(props.default - min) / (max - min) * 100}%` }"
      />
    </div>
    <div class="position-relative" style="width: 70px;">
      <input
        v-model.number="value"
        type="number"
        class="form-control form-control-sm"
        v-bind="props"
      >
      <span
        v-if="props.unit"
        class="position-absolute end-2 top-50 translate-middle-y text-muted small"
        style="pointer-events: none;"
      >
        {{ props.unit }}
      </span>
    </div>
  </div>
</template>

<style scoped>
.slider {
  appearance: none;
  height: 22px;
  outline: none;
  opacity: 0.7;
  transition: opacity .2s;
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