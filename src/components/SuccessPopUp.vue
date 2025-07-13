<template>
  <transition name="fade">
    <div
      v-if="visible"
      class="fixed top-3 right-6 bg-green-500 text-white px-4 py-3 rounded-lg shadow-lg flex items-center space-x-2 z-50"
    >
      <font-awesome-icon :icon="['fas', 'check-circle']" class="text-white" />
      <span>{{ message }}</span>
    </div>
  </transition>
</template>

<script setup>
defineProps({
  message: {
    type: String,
    required: true
  },
  visible: {
    type: Boolean,
    required: true
  }
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
