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
        idx ? 'border-start border-secondary ms-n1' : '',
        i === modelValue ? 'bg-secondary text-white' : '',
        'border-bottom'
      ]"
      :title="titles?.[idx]"
    >
      <div
        :class="[
          i === modelValue ? '' : 'opacity-35',
          titles?.[idx] ? '' : 'text-capitalize',
          classes?.[idx] || '',
        ]"
      >
        {{ titles?.[idx] ?? i }}
      </div>
      <input
        :value="i"
        @input="$emit('update:modelValue', $event.target.value)"
        type="radio"
        :title="titles?.[idx]"
        class="position-absolute inset-0 opacity-0"
      >
    </label>
  </fieldset>
</template>