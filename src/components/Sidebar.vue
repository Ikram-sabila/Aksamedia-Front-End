<template>
    <aside class="hidden md:block fixed top-0 left-0 h-screen w-48 bg-white border-r border-red-700 p-4 z-50">
        <RouterLink to="/dashboard" class="block text-center">
            <h2 class="text-2xl font-bold mb-6 text-red-700 hover:text-red-500 transition">
                <font-awesome-icon :icon="['fas', 'users-gear']" /> UN
            </h2>
        </RouterLink>
        <nav class="flex flex-col space-y-2 text-left">
            <RouterLink to="/dashboard" :class="[
                isActive('/dashboard')
                    ? 'bg-red-100 text-red-700 font-semibold rounded-xl'
                    : 'text-gray-600 hover:bg-gray-100 hover:text-red-600',
                'px-4 py-2 transition rounded-xl'
            ]">
                Dashboard
            </RouterLink>
            <RouterLink to="/profile" :class="[
                isActive('/profile')
                    ? 'bg-red-100 text-red-700 font-semibold rounded-xl'
                    : 'text-gray-600 hover:bg-gray-100 hover:text-red-600',
                'px-4 py-2 transition rounded-xl'
            ]">
                Profile
            </RouterLink>
        </nav>
    </aside>
</template>

<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()

const isActive = (path) => route.path === path
</script>