<template>
  <div v-if="visible" class="fixed inset-0 z-50 flex items-center justify-center bg-black/30 backdrop-blur-sm">
    <div class="bg-white w-full max-w-md p-6 rounded-2xl shadow-xl border border-red-500">
      <h2 class="text-xl font-semibold text-gray-800 mb-4 text-center">
        Konfirmasi Hapus
      </h2>

      <p class="text-gray-600 text-center mb-6">
        Apakah kamu yakin ingin menghapus <span class="font-bold">{{ karyawan.nama }}</span>?
      </p>

      <div class="flex justify-between space-x-2">
        <button
          @click="onCancel"
          type="button"
          class="px-4 py-2 bg-white border text-red-500 border-red-600 hover:bg-gray-200"
        >
          Batal
        </button>
        <button
          @click="onDelete"
          type="button"
          class="px-4 py-2 text-white bg-red-500 border-red-600 hover:bg-red-700 rounded-lg"
        >
          Hapus
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  visible: Boolean,
  karyawan: Object
})

const emit = defineEmits(['cancel', 'delete'])

const onCancel = () => emit('cancel')
const onDelete = () => emit('delete', props.karyawan.id)
</script>
