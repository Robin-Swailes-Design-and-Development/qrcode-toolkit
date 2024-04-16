<script setup lang="ts">
defineProps<{
  title: string
}>()

const value = defineModel<string>('modelValue', {
  type: String,
})

function clear() {
  value.value = ''
}
</script>

<template>
  <div
    class="relative flex flex-col items-center justify-center h-50 w-50 aspect-ratio-1 overflow-hidden border border-base rounded cursor-pointer"
    :class="value ? '' : 'border-dashed'"
    :hover="{ 'bg-gray-100': true }"
  >
    <img
      v-if="value"
      :src="value"
      class="absolute inset-0 aspect-ratio-1 rounded object-contain opacity-100"
    >
    <template v-else>
      <div class="text-lg opacity-50">
        <i class="ri-upload-line"></i>
      </div>
      <div class="text-center opacity-50">
        Upload
      </div>
    </template>
    <button
      v-if="value"
      class="absolute right-0 top-0 z-20 m-1 rounded-full bg-gray-200 p-1 opacity-50 hover:opacity-75"
      title="Remove image"
    >
      <div class="carbon-close" @click="clear"></div>
    </button>
    <ImageUpload v-model="value" />
  </div>
</template>