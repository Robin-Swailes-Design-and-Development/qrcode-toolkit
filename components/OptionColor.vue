<script setup lang="ts">
const value = defineModel<string>('modelValue', {
  type: String,
})
</script>

<template>
  <div
    class="relative flex items-center gap-2 bg-secondary p-0.5 px-1.5 border border-base rounded"
  >
    <div
      class="h-4 w-4 border border-base rounded-full"
      :style="{ background: value }"
    />
    <div class="text-sm font-mono">
      {{ value }}
    </div>
    <input
      v-model="value"
      type="color"
      class="absolute inset-0 z-10 opacity-0"
    >
  </div>
</template>