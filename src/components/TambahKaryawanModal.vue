<template>
  <div v-if="visible" class="fixed inset-0 z-50 flex items-center justify-center bg-black/30 backdrop-blur-sm">
    <div class="bg-white w-full max-w-md p-6 rounded-2xl shadow-xl border border-red-500">
      <h2 class="text-2xl font-semibold text-gray-800 mb-6 text-center">Tambah Karyawan</h2>

      <form @submit.prevent="handleAdd" class="space-y-5">
        <!-- Nama -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Nama</label>
          <input v-model="form.name" type="text"
            class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500" required />
        </div>

        <!-- Image -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Image</label>
          <input type="file" accept="image/*" @change="handleFileChange"
            class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500" />
        </div>

        <!-- Divisi -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Divisi</label>
          <div class="relative z-50">
            <div @click="toggleDropdown"
              class="flex items-center justify-between px-4 py-2 bg-gray-100 border border-gray-300 rounded-lg cursor-pointer">
              <span>{{ selectedDivision?.name || 'Pilih divisi' }}</span>
              <svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" stroke-width="2"
                viewBox="0 0 24 24" stroke-linecap="round" stroke-linejoin="round">
                <path d="M19 9l-7 7-7-7" />
              </svg>
            </div>

            <ul v-if="dropdownOpen"
              class="absolute left-0 top-full mt-1 w-full bg-white border border-gray-300 rounded shadow-lg z-50 max-h-48 overflow-y-auto">
              <li v-for="division in divisions" :key="division.id" @click="selectDivision(division)"
                class="px-4 py-2 hover:bg-gray-100 cursor-pointer">
                {{ division.name }}
              </li>
            </ul>
          </div>
        </div>

        <!-- Posisi -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Posisi</label>
          <input v-model="form.position" type="text"
            class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500" required/>
        </div>

        <!-- Phone -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Phone</label>
          <input v-model="form.phone" type="phone"
            class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500" required/>
        </div>

        <!-- Tombol -->
        <div class="flex gap-3 pt-4 justify-between">
          <button type="button" @click="onCancel"
            class="px-4 py-2 bg-white border text-red-500 border-red-600 rounded-lg hover:bg-red-50 transition">
            Batal
          </button>
          <button type="submit"
            class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-700 transition">
            Simpan
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref, watch } from 'vue'
import api from '../services/axios'

const props = defineProps(['visible'])
const emit = defineEmits(['add', 'cancel'])

const dropdownOpen = ref(false)
const toggleDropdown = () => {
  dropdownOpen.value = !dropdownOpen.value
}

const divisions = ref([])
const selectedDivision = ref(null)

const fetchDivisions = async () => {
  try {
    const response = await api.get('/divisions')
    divisions.value = response.data.data.divisions
  } catch (err) {
    console.log('Gagal mengambil data divisi: ', err)
  }
}

onMounted(fetchDivisions)

const selectDivision = (division) => {
  selectedDivision.value = division
  dropdownOpen.value = false
}

const form = reactive({
  id: null,
  name: '',
  image: null,
  phone: '',
  position: '',
  division: ''
})

watch(() => props.visible, (v) => {
  if (v) {
    form.name = ''
    form.image = null
    form.position = ''
    form.phone = ''
    selectedDivision.value =  null
  }
}, { immediate: true })

const handleAdd = () => {
  emit('add', {
    id: form.id,
    name: form.name,
    image: form.image,            
    position: form.position,     
    phone: form.phone,
    division_id: selectedDivision.value?.id ?? null 
  })
}

const handleFileChange = (event) => {
  const file = event.target.files[0]
  if (file) {
    form.image = file
  }
}

const onCancel = () => {
  emit('cancel')
}
</script>
