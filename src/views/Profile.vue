<template>
  <div class="flex flex-col md:flex-row h-screen bg-gray-100">
    <Sidebar class="hidden md:block"/>
    <div class="flex-1 flex flex-col md:ml-48">
      <Navbar />
      <div class="flex-1 overflow-y-auto px-8 py-10">
        <div class="bg-white shadow rounded-xl px-6 sm:px-10 py-3 sm:py-4 my-3 flex justify-between items-center">
          <div class="w-full">
            <h2 class="text-2xl font-semibold text-gray-800 mb-6">
              <font-awesome-icon :icon="['fas', 'user']" /> Ubah Profil
            </h2>

            <form @submit.prevent="simpanPerubahan" class="space-y-6">
              <div class="grid grid-cols-1 sm:grid-cols-12 items-center gap-4">
                <label class="col-span-1 sm:col-span-3 text-sm font-medium text-gray-700">Nama</label>
                <input v-model="form.nama" type="text"
                  class="col-span-1 sm:col-span-9 border border-gray-300 rounded-lg px-4 py-2 w-full focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  required />
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-12 items-center gap-4">
                <label class="col-span-1 sm:col-span-3 text-sm font-medium text-gray-700">Username</label>
                <input v-model="form.username" type="text"
                  class="col-span-1 sm:col-span-9 border border-gray-300 rounded-lg px-4 py-2 w-full focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  required />
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-12 items-center gap-4">
                <label class="col-span-1 sm:col-span-3 text-sm font-medium text-gray-700">Email</label>
                <input v-model="form.email" type="email"
                  class="col-span-1 sm:col-span-9 border border-gray-300 rounded-lg px-4 py-2 w-full focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  required />
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-12 items-center gap-4">
                <label class="col-span-1 sm:col-span-3 text-sm font-medium text-gray-700">Phone</label>
                <input v-model="form.phone" type="text"
                  class="col-span-1 sm:col-span-9 border border-gray-300 rounded-lg px-4 py-2 w-full focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  required />
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-12 items-center gap-4">
                <label class="col-span-1 sm:col-span-3 text-sm font-medium text-gray-700">Verifikasi Password</label>
                <div class="col-span-1 sm:col-span-9 relative">
                  <input :type="showPassword ? 'text' : 'password'" v-model="form.password"
                    class="w-full border border-gray-300 rounded-lg px-4 py-2 pr-10 focus:ring-2 focus:ring-blue-500 focus:outline-none"
                    placeholder="Masukkan password" />
                  <button type="button" @click="togglePassword"
                    class="absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-500 hover:text-gray-700">
                    <font-awesome-icon :icon="['fas', showPassword ? 'eye-slash' : 'eye']" />
                  </button>
                </div>
              </div>

              <div class="flex justify-between items-center gap-4 pt-2">
                <RouterLink to="/dashboard"
                  class="inline-block px-5 py-2 bg-white text-red-600 rounded-xl hover:bg-red-50 transition border border-red-500 text-sm">
                  Batal
                </RouterLink>
                <button type="submit"
                  class="inline-block px-5 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition">
                  Simpan
                </button>
              </div>
            </form>

          </div>
        </div>
      </div>
    </div>
  </div>
  <SuccessPopUp v-if="showSuccess" :message="successMessage" :visible="showSuccess" />
  <ErrorPopUp v-if="showError" :visible="showError" :message="errorMessage" :errors="errorDetails"
      @close="showError = false" />
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import Sidebar from '../components/Sidebar.vue'
import Navbar from '../components/Navbar.vue'
import api from '../services/axios'
import SuccessPopUp from '../components/SuccessPopUp.vue'
import ErrorPopUp from '../components/ErrorPopUp.vue'

const router = useRouter()
const showPassword = ref(false)
const showSuccess = ref(false)
const successMessage = ref('')

const showError = ref(false)
const errorMessage = ref('')
const errorDetails = ref({})

const form = reactive({
  nama: '',
  username: '',
  email: '',
  phone: '',
  password: ''
})

const togglePassword = () => {
  showPassword.value = !showPassword.value
}

onMounted(() => {
  const user = JSON.parse(localStorage.getItem('user'))
  if (user) {
    form.nama = user.name
    form.username = user.username
    form.email = user.email
    form.phone = user.phone
  }
})

const simpanPerubahan = async () => {
  try {
    const payload = {
      name: form.nama,
      username: form.username,
      email: form.email,
      phone: form.phone
    }

    if (form.password) {
      payload.password = form.password
    }

    const response = await api.put('/user/update', payload)

    const updatedUser = response.data.data.user
    localStorage.setItem('user', JSON.stringify(updatedUser))

    successMessage.value = response.data.message || 'Profil berhasil diperbarui!'
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
</script>
