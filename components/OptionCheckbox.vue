<script setup lang="ts">
const value = defineModel<boolean>('modelValue', {
  type: Boolean,
})
</script>

<template>
  <div
    class="relative flex items-center gap-2 h-5 w-5 p-0.5 border border-base rounded cursor-pointer"
  >
    <div
      class="i-ri-check-line"
      :class="value ? '' : 'opacity-0'"
    />
    <input
      v-model="value"
      type="checkbox"
      class="absolute inset-0 z-10 opacity-0"
    >
  </div>
</template>