<script setup>
defineProps({
  id: String,
  title: String,
  show: Boolean,
});
</script>

<template>
  <div
    class="offcanvas offcanvas-end"
    :class="{ show }"
    tabindex="-1"
    :id="id"
    data-bs-backdrop="false"
    aria-labelledby="offcanvas"
  >
    <div class="offcanvas-header border-bottom">
      <h5 class="offcanvas-title" id="offcanvas" v-text="title" />
      <button
        type="button"
        class="btn-close"
        data-bs-dismiss="offcanvas"
        aria-label="Close"
      ></button>
    </div>
    <div class="offcanvas-body py-4">
      <slot />
    </div>
  </div>
</template>
