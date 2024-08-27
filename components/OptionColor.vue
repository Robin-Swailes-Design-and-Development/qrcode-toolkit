<template>
  <div class="position-relative d-flex align-items-center gap-2 p-1 px-2 border rounded">
    <div
      class="color-pick border rounded-circle"
      :style="{ background: modelValue }"
    ></div>
    <div class="small font-monospace">
      {{ modelValue }}
    </div>
    <input
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      type="color"
      class="position-absolute inset-0 opacity-0"
      style="z-index: 10;"
    >
  </div>
</template>

<script>
export default {
  props: {
    modelValue: {
      type: String,
      required: true
    }
  },
  emits: ['update:modelValue']
}
</script>

<style scoped>
.color-pick {
  width: 30px;
  height: 30px;
}
</style>