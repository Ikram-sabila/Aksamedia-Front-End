<template>
  <div class="min-h-screen w-full flex items-center justify-center bg-blue-50 p-4">
    <div class="flex flex-col md:flex-row w-full max-w-5xl bg-white rounded-lg shadow-lg overflow-hidden">
      <!-- Left side -->
      <div class="hidden md:flex md:w-1/2 bg-[#DC3C22] items-center justify-center">
        <div class="text-white text-center p-8">
          <h2 class="text-3xl font-bold mb-4">Selamat Datang Kembali!</h2>
          <p class="text-lg">Masuk untuk mengakses dashboard dan fitur menarik lainnya.</p>
        </div>
      </div>

      <!-- Right side -->
      <div class="w-full md:w-1/2 p-6 sm:p-8">
        <h2 class="text-2xl font-semibold text-center text-gray-700 mb-6">Login Akun</h2>
        <form @submit.prevent="handleLogin" class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-600 mb-1">Username</label>
            <input
              v-model="username"
              type="text"
              placeholder="contohusername"
              class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
              required
            />
          </div>

          <div class="relative">
            <label class="block text-sm font-medium text-gray-600 mb-1">Password</label>
            <div class="relative flex items-center">
              <input
                :type="showPassword ? 'text' : 'password'"
                v-model="password"
                placeholder="••••••••"
                class="w-full border border-gray-300 rounded-lg px-3 py-2 pr-10 focus:outline-none focus:ring-2 focus:ring-blue-500"
                required
              />
              <button
                type="button"
                @click="togglePassword"
                class="absolute top-1/2 transform -translate-y-1/2 right-3 text-gray-500 hover:text-gray-700"
              >
                <font-awesome-icon :icon="['fas', showPassword ? 'eye-slash' : 'eye']" />
              </button>
            </div>
          </div>

          <button
            type="submit"
            class="w-full bg-[#DC3C22] text-white py-2 rounded-lg hover:bg-white hover:text-[#DC3C22] hover:border hover:border-[#DC3C22] transition duration-200"
          >
            Login
          </button>

          <p v-if="error" class="text-red-500 text-sm text-center mt-2">{{ error }}</p>
        </form>

        <div class="text-sm text-center mt-6 text-gray-600">
          Belum punya akun?
          <a href="/register" class="hover:underline text-red-600">Daftar sekarang</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import api from '../services/axios'

const username = ref('')
const password = ref('')

const error = ref('')
const showPassword = ref(false)
const router = useRouter()

const handleLogin = async () => {
  try {
    const response = await api.post('/login', {
      username: username.value,
      password: password.value
    })
    localStorage.setItem('token', response.data.data.token)
    localStorage.setItem('user', JSON.stringify(response.data.data.admin))
    router.push('/dashboard')
  } catch (err) {
    error.value = err.response?.data?.message || 'Login gagal. Cek username atau password.'
  }
}

const togglePassword = () => {
  showPassword.value = !showPassword.value
}
</script>
