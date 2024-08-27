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
  <fieldset class="d-inline-flex flex-wrap overflow-hidden small border rounded">
    <label
      v-for="(i, idx) of options"
      :key="i"
      class="position-relative px-2 py-1 mb-n1 hover-bg-light"
      :class="[
        idx ? 'border-start ms-n1' : '',
        i === value ? 'bg-secondary text-white' : '',
        'border-bottom'
      ]"
      :title="titles?.[idx]"
    >
      <div
        :class="[
          i === value ? '' : 'opacity-50',
          titles?.[idx] ? '' : 'text-capitalize',
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
        class="position-absolute top-0 start-0 w-100 h-100 opacity-0"
      >
    </label>
  </fieldset>
</template>