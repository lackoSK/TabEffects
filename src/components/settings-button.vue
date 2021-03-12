<template>
  <button class="settings-button no-outline">
    <span class="settings-button-header">{{ header }}</span>
    <span class="settings-button-description">{{  description }}</span>
  </button>
</template>

<script>
export default {
  name: "settings-button",
  props: {
    header: {
      type: String,
      required: true,
    },
    description: {
      type: String,
      required: true,
    }
  }
}
</script>

<style scoped>
.settings-button {
  @apply block w-full text-base p-3 hover:bg-gray-50 transition rounded-md text-left;
}

.settings-button-header {
  @apply text-base font-medium text-gray-900 block;
}

.settings-button-description {
  @apply mt-1 text-sm text-gray-500;
}

</style>