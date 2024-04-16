<script setup lang="ts">
import { ref } from 'vue';

defineProps<{
  modelValue?: string | undefined
}>()

const emit = defineEmits<{
  (event: 'update:modelValue', dataurl: string): void
}>()

const value = ref<any>()

async function read(e: Event) {
  value.value = ''
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return
  const reader = new FileReader()
  const promise = new Promise<string>((resolve, reject) => {
    reader.onload = () => {
      resolve(reader.result as any)
    }
    reader.onerror = reject
  })
  reader.readAsDataURL(file)
  emit('update:modelValue', await promise)
}
</script>

<template>
  <input
    type="file"
    accept="image/*"
    :value="value"
    class="absolute inset-0 z-10 max-h-full max-w-full cursor-pointer opacity-0"
    @input="read"
  >
</template>