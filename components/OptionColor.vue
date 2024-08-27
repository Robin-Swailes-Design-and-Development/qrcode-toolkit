<script setup lang="ts">
const value = defineModel<string>('modelValue', {
  type: String,
})
</script>

<template>
  <div
    class="relative flex items-center gap-2 p-0.5 px-1.5 border border-base rounded"
  >
    <div
      class="color-pick border border-base rounded-full"
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



<style scoped>
.color-pick{
  width: 30px;
  height: 30px;
}

</style>