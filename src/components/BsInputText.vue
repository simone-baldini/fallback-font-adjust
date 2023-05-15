<script setup>
import { computed } from "vue";

const props = defineProps({
  id: String,
  label: String,
  placeholder: String,
  modelValue: String,
});

const range = computed(() => {
  return props.max - props.min;
});

const progress = computed(() => {
  debugger;
  return ((parseInt(props.modelValue, 10) - props.min) * 100) / range.value;
});

const offset = computed(() => {
  return (
    10 -
    (((parseInt(props.modelValue, 10) - props.min) * 100) / range.value) * 0.2
  );
});
</script>

<template>
  <div class="form-floating">
    <input
      type="text"
      class="form-control rounded-pill px-4"
      :id="id"
      :placeholder="placeholder"
      @input="$emit('update:modelValue', $event.target.value)"
    />
    <label :for="id" class="opacity-75 px-4" v-text="label" />
  </div>
</template>

<style lang="scss" scoped>
.rangeHint {
  min-width: 5rem;
  text-align: center;
}

.form-range {
  position: relative;
  padding: 1.625rem 0 0.625rem;
  height: calc(3.5rem + 2px);
  line-height: 1.25;
  label {
    transform: scale(0.85) translateY(-0.5rem) translateX(0.15rem);
    transform-origin: 0 0;
    position: absolute;
    top: 0;
    left: 0;
    padding: 1rem 0;
  }
  input[type="range"] {
    width: 100%;
  }
  span {
    content: attr(data-value);
    position: absolute;
    top: -0.75rem;
    padding: 0.5rem 0.75rem;
    font-size: 80%;
    background-color: #1a73e8;
    color: #fff;
    border-radius: 50rem;
    left: 148px;
    transform: scale(0) translateX(-50%);
    transform-origin: left;
    transition: transform 0.2s ease-in-out, opacity 0.2s ease-in-out;
    opacity: 0;
    left: calc(var(--progress) + var(--offset));
    &::before {
      content: " ";
      width: 0;
      height: 0;
      border-left: 4px solid transparent;
      border-right: 4px solid transparent;
      border-top: 4px solid #1a73e8;
      position: absolute;
      bottom: 0;
      left: 50%;
      transform: translate(-50%, 100%);
    }
  }
  &:focus-within {
    span {
      opacity: 1;
      transform: scale(1) translateX(-50%);
    }
  }
}

input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  background-color: #d2e3fc;
  height: 0.25rem;
  border-radius: 50rem;
  cursor: pointer;
}

input[type="range"]::-webkit-slider-runnable-track {
  border-radius: 50rem;
  background-image: linear-gradient(#1a73e8, #1a73e8);
  background-size: var(--progress) 100%;
  background-repeat: no-repeat;
  height: calc(0.25rem + 2px);
  margin: -1px 0;
}

input[type="range"]::-moz-range-track {
  border-radius: 50rem;
  background-image: linear-gradient(#1a73e8, #1a73e8);
  background-size: var(--progress) 100%;
  background-repeat: no-repeat;
  height: calc(0.25rem + 2px);
  margin: -1px 0;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none; /* Override default look */
  appearance: none;
  margin-top: -0.5rem; /* Centers thumb on the track */
  background-color: #1a73e8;
  height: 1.25rem;
  width: 1.25rem;
  border-radius: 50%;
}

input[type="range"]::-moz-range-thumb {
  border: none; /*Removes extra border that FF applies*/
  background-color: #1a73e8;
  height: 1.25rem;
  width: 1.25rem;
  border-radius: 50%;
}

input[type="range"]:focus {
  outline: none;
}

input[type="range"]:focus::-webkit-slider-thumb {
  box-shadow: 0 0 0 0.5rem rgba(13, 110, 253, 0.25);
}

input[type="range"]:focus::-moz-range-thumb {
  box-shadow: 0 0 0 0.5rem rgba(13, 110, 253, 0.25);
}
</style>
