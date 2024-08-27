<template>
  <div class="position-relative d-flex align-items-center gap-2 p-1 px-2 border rounded">
    <div
      class="color-pick border rounded-circle"
      :style="{ background: modelValue }"
    ></div>
    <input
      :value="modelValue"
      @input="updateColor($event.target.value)"
      type="text"
      class="form-control form-control-sm"
      style="width: 100px;"
      placeholder="#RRGGBB"
    >
    <input
      :value="modelValue"
      @input="updateColor($event.target.value)"
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
  emits: ['update:modelValue'],
  methods: {
    updateColor(value) {
      // Validate the color input
      if (/^#[0-9A-Fa-f]{6}$/.test(value)) {
        this.$emit('update:modelValue', value);
      } else if (value === '') {
        // Allow empty input to clear the color
        this.$emit('update:modelValue', '');
      }
    }
  }
}
</script>

<style scoped>
.color-pick {
  width: 30px;
  height: 30px;
}
</style>