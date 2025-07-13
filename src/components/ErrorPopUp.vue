<template>
  <div v-if="visible" class="fixed inset-0 z-50 flex items-center justify-center bg-black/30 backdrop-blur-sm">
    <div class="bg-white w-full max-w-md p-6 rounded-2xl shadow-xl border border-red-500">
      <h2 class="text-xl font-semibold text-red-600 mb-4 text-center">Terjadi Kesalahan</h2>

      <p class="text-gray-800 text-sm mb-3">{{ message }}</p>

      <ul v-if="errors && Object.keys(errors).length" class="list-disc list-inside text-sm text-red-500 space-y-1">
        <li v-for="(errorMessages, field) in errors" :key="field">
          <span v-for="msg in errorMessages" :key="msg">{{ msg }}</span>
        </li>
      </ul>

      <div class="flex justify-end pt-4">
        <button @click="$emit('close')"
          class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition text-sm">
          Tutup
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  visible: Boolean,
  message: String,
  errors: Object
})
defineEmits(['close'])
</script>
