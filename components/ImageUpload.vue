<script setup lang="ts">
import { ref } from 'vue';

defineProps<{
  modelValue?: string | undefined
}>();

const emit = defineEmits<{
  (event: 'update:modelValue', dataurl: string): void
}>();

const value = ref<any>();

async function read(e: Event) {
  value.value = '';
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;
  const reader = new FileReader();
  const promise = new Promise<string>((resolve, reject) => {
    reader.onload = () => {
      resolve(reader.result as any);
    };
    reader.onerror = reject;
  });
  reader.readAsDataURL(file);
  emit('update:modelValue', await promise);
}
</script>

<template>
  <div class="position-relative" style="width: 100px; height: 100px;">
    <input
      type="file"
      accept="image/*"
      :value="value"
      class="position-absolute top-0 start-0 w-100 h-100 opacity-0 cursor-pointer"
      @input="read"
    >
    <div class="w-100 h-100 border rounded d-flex align-items-center justify-content-center bg-light">
      <i class="bi-upload fs-3"></i>
    </div>
  </div>
</template>