<script setup>
defineProps({
  id: String,
  label: String,
  modelValue: [String],
  placeholder: String,
  ariaLabel: String,
  options: [Object, Array],
  disabled: {
    type: Boolean,
    default: false,
  },
});
</script>

<template>
  <div class="form-floating">
    <select
      class="form-select rounded-pill ps-4 pe-5"
      :id="id"
      :aria-label="label"
      :disabled="disabled"
      @input="$emit('update:modelValue', $event.target.value)"
    >
      <option disabled value="" v-text="placeholder" />
      <slot />
    </select>
    <label :for="id" class="opacity-75 ps-4 pe-5" v-text="label" />
  </div>
</template>
