<script setup lang="ts">
defineProps<{
  options: readonly string[] | number[]
  titles?: string[]
  classes?: string[]
}>()

const value = defineModel<string | number>('modelValue', {
  type: [String, Number],
})
</script>

<template>
  <fieldset class="flex flex-wrap inline-flex overflow-hidden text-xs border border-base rounded">
    <label
      v-for="(i, idx) of options"
      :key="i"
      class="relative px-2 py-1 mb--1px hover:bg-offwhite"
      :class="[
        idx ? 'border-l border-base ml--1px' : '',
        i === value ? 'bg-nopsslate text-white' : '',
        'border-b border-base',
      ]"
      :title="titles?.[idx]"
    >
      <div
        :class="[
          i === value ? '' : 'opacity-35',
          titles?.[idx] ? '' : 'capitalize',
          classes?.[idx] || '',
        ]"
      >
        {{ titles?.[idx] ?? i }}
      </div>
      <input
        v-model="value"
        type="radio"
        :value="i"
        :title="titles?.[idx]"
        class="absolute inset-0 opacity-0"
      >
    </label>
  </fieldset>
</template>