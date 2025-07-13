<template>
  <div class="flex flex-col md:flex-row h-screen">
    <Sidebar class="hidden md:block" />

    <div class="flex-1 flex flex-col md:ml-48">
      <Navbar />

      <div class="p-4 sm:p-6">
        <h2 class="text-xl sm:text-2xl font-bold mb-6 flex items-center gap-2">
          <font-awesome-icon :icon="['fas', 'people-group']" /> Daftar Karyawan
        </h2>

        <div class="flex flex-col sm:flex-row sm:items-center gap-3 sm:gap-0 mb-4">
          <div class="flex items-center border border-gray-300 rounded-xl px-3 py-2 w-full sm:flex-[auto]">
            <font-awesome-icon :icon="['fas', 'magnifying-glass']" class="text-gray-500 mr-2" />
            <input v-model="searchQuery" type="text" placeholder="Cari karyawan..." class="w-full focus:outline-none" />
          </div>

          <div class="relative w-full sm:w-48 sm:ml-4">
            <div @click="toggleDropDown"
              class="flex items-center justify-between px-4 py-2 border border-gray-300 rounded-xl cursor-pointer">
              <span class="truncate">{{ selectedDivision?.name || 'All Divisions' }}</span>
              <svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"
                stroke-linecap="round" stroke-linejoin="round">
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


        <div class="overflow-x-auto bg-white shadow rounded-2xl border border-gray-300">
          <table class="min-w-full divide-y divide-gray-200 text-sm">
            <thead>
              <tr>
                <th colspan="6"
                  class="text-center py-4 text-gray-500 border-b border-dashed border-gray-200 bg-gray-100">
                  <div class="flex justify-between items-center px-4">
                    <h5 class="text-lg font-semibold text-black">Karyawan</h5>
                    <button @click="tambahKaryawan()"
                      class="bg-green-500 hover:bg-green-600 text-white text-sm px-3 py-1 rounded-sm">
                      Add <font-awesome-icon :icon="['fas', 'plus']" />
                    </button>
                  </div>
                </th>
              </tr>
              <tr class="bg-[#FCEF91]">
                <th class="px-4 py-2 text-left">ID</th>
                <th class="px-4 py-2 text-left">Nama</th>
                <th class="px-4 py-2 text-left">Divisi</th>
                <th class="px-4 py-2 text-left">Posisi</th>
                <th class="px-4 py-2 text-left">Phone</th>
                <th class="px-4 py-2 text-center">Aksi</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="karyawan in paginatedKaryawans" :key="karyawan.id" class="border-t">
                <td class="px-4 py-2">{{ karyawan.id }}</td>
                <td class="px-4 py-2">{{ karyawan.name }}</td>
                <td class="px-4 py-2">{{ karyawan.division.name }}</td>
                <td class="px-4 py-2">{{ karyawan.position }}</td>
                <td class="px-4 py-2">{{ karyawan.phone }}</td>
                <td class="px-4 py-2 text-center">
                  <button @click="detailKaryawan(karyawan.id)" class="text-xs text-blue-600 hover:underline">
                    <font-awesome-icon :icon="['fas', 'circle-info']" />
                  </button>
                </td>
              </tr>
              <tr v-if="paginatedKaryawans.length === 0">
                <td colspan="6" class="text-center py-4 text-gray-500">Tidak ada data karyawan</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="mt-4 flex flex-col sm:flex-row justify-between items-center gap-2">
          <button @click="currentPage--" :disabled="currentPage === 1"
            class="px-4 py-2 bg-white rounded hover:bg-red-200 disabled:opacity-50 text-red-800 border border-red-600">
            Sebelumnya
          </button>
          <span>Halaman {{ currentPage }} dari {{ totalPages }}</span>
          <button @click="currentPage++" :disabled="currentPage === totalPages"
            class="px-4 py-2 bg-white rounded hover:bg-red-200 disabled:opacity-50 text-red-800 border border-red-600">
            Selanjutnya
          </button>
        </div>
      </div>
    </div>

    <TambahKaryawanModal :visible="modalAddVisible" @add="onAdd" @cancel="modalAddVisible = false" />
    <SuccessPopUp v-if="showSuccess" :message="successMessage" :visible="showSuccess" />
    <ErrorPopUp v-if="showError" :visible="showError" :message="errorMessage" :errors="errorDetails"
      @close="showError = false" />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import Sidebar from '../components/Sidebar.vue'
import Navbar from '../components/Navbar.vue'
import api from '../services/axios'
import router from '../router'
import { useRoute } from 'vue-router'

import TambahKaryawanModal from '../components/TambahKaryawanModal.vue'
import SuccessPopUp from '../components/SuccessPopUp.vue'
import ErrorPopUp from '../components/ErrorPopUp.vue'

const route = useRoute()
const searchQuery = ref('')
const currentPage = ref(1)

const modalAddVisible = ref(false)

const karyawans = ref([])
const totalPages = ref(1)

const dropdownOpen = ref(false)
const toggleDropDown = () => {
  dropdownOpen.value = !dropdownOpen.value
}

const divisions = ref([])
const selectedDivision = ref(null)

const showSuccess = ref(false)
const successMessage = ref('')

const showError = ref(false)
const errorMessage = ref('')
const errorDetails = ref({})

const fetchDivisions = async () => {
  try {
    const response = await api.get('/divisions')
    divisions.value = [
      { id: null, name: 'All Divisions' },
      ...response.data.data.divisions
    ]
  } catch (err) {
    console.log('Gagal mengambil data divisi: ', err)
  }
}

onMounted(fetchDivisions)

const selectDivision = (division) => {
  selectedDivision.value = division
  dropdownOpen.value = false
  fetchKaryawan()
}

const fetchKaryawan = async () => {
  try {
    const res = await api.get('/employees', {
      params: {
        name: searchQuery.value || null,
        page: currentPage.value,
        division_id: selectedDivision.value?.id || null
      }
    })

    karyawans.value = res.data.data.employees
    const pagination = res.data.pagination
    totalPages.value = pagination.last_page
  } catch (err) {
    console.error('Gagal mengambil data karyawan:', err)
  }
}

onMounted(() => {
  const pageFromQuery = parseInt(route.query.page) || 1
  currentPage.value = pageFromQuery

  const searchFromQuery = route.query.search || ''
  searchQuery.value = searchFromQuery

  fetchKaryawan()
})

watch(currentPage, (newPage) => {
  router.replace({ query: { ...route.query, page: newPage } })
})

let debounceTimer
watch([searchQuery, currentPage], () => {
  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => {
    router.replace({
      query: {
        ...route.query,
        search: searchQuery.value || undefined,
        page: currentPage.value,
      }
    })
    fetchKaryawan()
  }, 1000)
})

const paginatedKaryawans = computed(() => karyawans.value)

const detailKaryawan = (id) => {
  router.push(`/employee/${id}`)
}

const tambahKaryawan = () => {
  modalAddVisible.value = true
}

const onAdd = async (addData) => {
  try {
    const formData = new FormData()
    formData.append('name', addData.name)
    formData.append('position', addData.position)
    formData.append('phone', addData.phone)
    formData.append('division_id', addData.division_id)
    if (addData.image) {
      formData.append('image', addData.image)
    }

    const response = await api.post(`/employees/`, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })

    modalAddVisible.value = false

    successMessage.value = response.data.message || 'Karyawan berhasil ditambahkan!'
    showSuccess.value = true

    setTimeout(() => {
      showSuccess.value = false
    }, 3000)
  } catch (err) {
    showError.value = true
    // console.log('Full error response:', err.response?.data)
    if (err.response && err.response.status === 422) {
      errorMessage.value = err.response.data.message || 'Validasi gagal'
      errorDetails.value = err.response.data.error || {}
    } else {
      errorMessage.value = err.response?.data?.message || 'Terjadi kesalahan pada server'
      errorDetails.value = {}
    }
  }
}

// const testErrorPopUp = () => {
//   errorMessage.value = 'Gagal menyimpan data!'
//   errorDetails.value = {
//     name: ['Nama wajib diisi'],
//     phone: ['Nomor telepon tidak valid'],
//   }
//   showError.value = true
// }
</script>


<!-- <script setup>
import { ref, computed } from 'vue'
import Sidebar from '../components/Sidebar.vue'
import Navbar from '../components/Navbar.vue'
import EditKaryawanModal from '../components/EditKaryawanModal.vue'
import HapusKaryawanModel from '../components/HapusKaryawanModel.vue'

const searchQuery = ref('')
const currentPage = ref(1)
const itemsPerPage = 5

const modalEditVisible = ref(false)
const modalDeleteVisible = ref(false)
const selectedKaryawan = ref(null)

const karyawans = ref([
  { id: 1, nama: 'Ikram', jabatan: 'Data Saintis', email: 'ikram@example.com' },
  { id: 2, nama: 'Navissa', jabatan: 'Keperawatan', email: 'nana@example.com' },
  { id: 3, nama: 'Sita', jabatan: 'Dokter', email: 'sita@example.com' },
  { id: 4, nama: 'Budi', jabatan: 'Frontend Developer', email: 'budi@example.com' },
  { id: 5, nama: 'Lisa', jabatan: 'HR', email: 'lisa@example.com' },
  { id: 6, nama: 'Doni', jabatan: 'Marketing', email: 'doni@example.com' },
  { id: 7, nama: 'Tina', jabatan: 'Keuangan', email: 'tina@example.com' },
])

const filteredKaryawans = computed(() => {
  if (!searchQuery.value) return karyawans.value
  return karyawans.value.filter((k) =>
    k.nama.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    k.jabatan.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    k.email.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const totalPages = computed(() =>
  Math.ceil(filteredKaryawans.value.length / itemsPerPage)
)

const paginatedKaryawans = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredKaryawans.value.slice(start, start + itemsPerPage)
})

const editKaryawan = (id) => {
  const found = karyawans.value.find(k => k.id === id)
  if (found) {
    selectedKaryawan.value = { ...found }
    modalEditVisible.value = true
  }
}

const showModalHapus = (id) => {
  const found = karyawans.value.find(k => k.id === id)
  if (found) {
    selectedKaryawan.value = { ...found }
    modalDeleteVisible.value = true
  }
}

const onUpdate = (updated) => {
  const index = karyawans.value.findIndex(k => k.id === updated.id)
  if (index !== -1) {
    karyawans.value[index] = updated
  }
  modalEditVisible.value = false
}

const onDelete = (id) => {
  karyawans.value = karyawans.value.filter((k) => k.id !== id)
  modalDeleteVisible.value = false
}

const tambahKaryawan = () => {
  alert('Fitur tambah belum diimplementasikan.')
}
</script> -->
