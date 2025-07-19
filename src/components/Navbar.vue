<template>
  <header class="w-full bg-white shadow px-6 py-4 flex justify-between items-center">
    <div>
      <RouterLink
        to="/dashboard"
        class="block md:hidden text-red-700 hover:text-red-500 transition text-xl font-bold"
      >
        <font-awesome-icon :icon="['fas', 'users-gear']" /> UN
      </RouterLink>

      <h2 class="hidden md:block text-2xl font-bold text-gray-800">
        {{ pageTitle }}
      </h2>
    </div>

    <div class="flex items-center">
      <span class="text-gray-700 font-medium px-1">{{ username }}</span>
      <div class="relative" @click="toggleDropdown">
        <button class="flex items-center space-x-2 focus:outline-none ml-2">
          <font-awesome-icon :icon="['fas', 'bars']" />
        </button>

        <div
          v-if="dropdownOpen"
          class="absolute right-0 mt-2 bg-white border rounded shadow z-10 min-w-full text-center"
        >
          <button
            @click="dashboard"
            class="text-left px-4 py-2 text-sm hover:bg-gray-100 min-w-full"
          >
            <font-awesome-icon :icon="['fas', 'chart-line']" /> Dashboard
          </button>
          <button
            @click="profil"
            class="text-left px-4 py-2 text-sm hover:bg-gray-100 min-w-full"
          >
            <font-awesome-icon :icon="['fas', 'user-tie']" /> Profil
          </button>
          <button
            @click="logout"
            class="text-left px-4 py-2 text-sm hover:bg-gray-100 min-w-full"
          >
            <font-awesome-icon :icon="['fas', 'arrow-right-from-bracket']" /> Logout
          </button>
        </div>
      </div>
    </div>
  </header>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import api from '../services/axios' 

const dropdownOpen = ref(false)
const username = ref('')
const router = useRouter()
const route = useRoute()

const toggleDropdown = () => {
    dropdownOpen.value = !dropdownOpen.value
}

const profil = () => {
    router.push('/profile')
}

const logout = async () => {
    try {
        await api.post('/logout')
    } catch (err) {
        console.error('Gagal logout dari server:', err.response?.data || err.message)
    } finally {
        localStorage.removeItem('token')
        localStorage.removeItem('user')
        router.push('/login')
    }
}

onMounted(() => {
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
        try {
            const user = JSON.parse(storedUser);
            username.value = user.name || 'Pengguna';
        } catch (e) {
            username.value = 'Pengguna';
        }
    } else {
        username.value = 'Pengguna';
    }
})

const pageTitle = computed(() => route.name || 'Dashboard')
const dashboard = () => {
  router.push('/dashboard')
}
</script>
