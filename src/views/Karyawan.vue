<template>
    <div class="flex flex-col md:flex-row h-screen bg-gray-100">
        <Sidebar class="hidden md:block" />

        <div class="flex-1 flex flex-col md:ml-48">
            <Navbar />

            <div class="flex justify-center mt-10 px-4">
                <div class="w-full max-w-5xl">
                    <div
                        class="bg-white shadow rounded-xl px-4 sm:px-10 py-3 sm:py-4 my-3 flex flex-row sm:flex-row justify-between items-start sm:items-center gap-2">
                        <h3 class="text-base sm:text-lg font-semibold text-gray-800">Data Karyawan</h3>
                        <div class="space-x-2">
                            <button @click="editKaryawan(karyawan.id)" class="text-xs">
                                <font-awesome-icon :icon="['fas', 'pen-to-square']" class="text-yellow-500 text-sm" />
                            </button>
                            <button @click="deleteKaryawan(karyawan.id)" class="text-xs">
                                <font-awesome-icon :icon="['fas', 'trash-can']" class="text-red-500 text-sm" />
                            </button>
                        </div>
                    </div>

                    <div v-if="karyawan" class="w-full bg-white shadow rounded-xl p-4 sm:p-6">
                        <div class="flex flex-col md:flex-row gap-6 md:gap-10 items-center md:items-start">
                            <div
                                class="w-32 h-32 sm:w-40 sm:h-40 rounded-full overflow-hidden border-2 border-blue-300 shadow">
                                <img :src="getImageUrl(karyawan.image) || `https://ui-avatars.com/api/?name=${encodeURIComponent(karyawan.name)}&background=0D8ABC&color=fff&rounded=true&size=256`"
                                    alt="Foto Karyawan" class="w-full h-full object-cover" />
                            </div>

                            <div class="flex-1 w-full space-y-2 text-sm text-gray-700">
                                <h2 class="text-xl sm:text-2xl font-bold text-gray-800 mb-2">{{ karyawan.name }}</h2>

                                <div v-if="karyawan.id" class="flex justify-between border-b py-1">
                                    <span class="font-medium text-gray-600">ID</span>
                                    <span>{{ karyawan.id }}</span>
                                </div>

                                <div v-if="karyawan.position" class="flex justify-between border-b py-1">
                                    <span class="font-medium text-gray-600">Posisi</span>
                                    <span>{{ karyawan.position }}</span>
                                </div>

                                <div v-if="karyawan.division?.name" class="flex justify-between border-b py-1">
                                    <span class="font-medium text-gray-600">Divisi</span>
                                    <span>{{ karyawan.division.name }}</span>
                                </div>

                                <div v-if="karyawan.phone" class="flex justify-between border-b py-1">
                                    <span class="font-medium text-gray-600">No. Telepon</span>
                                    <span>{{ karyawan.phone }}</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Loading State -->
                    <div v-else class="text-center mt-20 text-gray-600 text-lg">
                        Memuat data karyawan...
                    </div>
                </div>
            </div>
        </div>
    </div>

    <EditKaryawanModal :visible="modalEditVisible" :karyawan="karyawan" @update="onUpdate"
        @cancel="modalEditVisible = false" />
    <HapusKaryawanModel :visible="modalDeleteVisible" :karyawan="karyawan" @delete="onDelete"
        @cancel="modalDeleteVisible = false" />
    <SuccessPopUp v-if="showSuccess" :message="successMessage" :visible="showSuccess" />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import Sidebar from '../components/Sidebar.vue'
import Navbar from '../components/Navbar.vue'
import api from '../services/axios'
import router from '../router'

import EditKaryawanModal from '../components/EditKaryawanModal.vue'
import HapusKaryawanModel from '../components/HapusKaryawanModel.vue'
import SuccessPopUp from '../components/SuccessPopUp.vue'

const route = useRoute()
const defaultImage = '/images/default-profile.png'

const modalEditVisible = ref(false)
const modalDeleteVisible = ref(false)

const showSuccess = ref(false)
const successMessage = ref('')

const editKaryawan = () => {
    modalEditVisible.value = true
}

const deleteKaryawan = () => {
    modalDeleteVisible.value = true
}

const onUpdate = async (updatedData) => {
    try {
        const formData = new FormData()
        formData.append('name', updatedData.name)
        formData.append('position', updatedData.position)
        formData.append('phone', updatedData.phone)
        formData.append('division_id', updatedData.division_id)
        if (updatedData.image) {
            formData.append('image', updatedData.image)
        }

        const response = await api.post(`/employees/${updatedData.id}?_method=PUT`, formData, {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
        })

        fetchKaryawan()
        modalEditVisible.value = false

        successMessage.value = response.data.message
        showSuccess.value = true

        setTimeout(() => {
            showSuccess.value = false
        }, 3000)
    } catch (err) {
        console.error('Gagal mengupdate data: ', err)
    }
}

const onDelete = async (id) => {
    try {
        await api.delete(`/employees/${id}`)
        router.push('/dashboard')
    } catch (err) {
        console.error('Gagal menghapus data: ', err)
    }
}

const karyawan = ref(null)

const getImageUrl = (path) => {
    if (!path) return null
    const baseUrl = import.meta.env.VITE_API_URL.replace(/\/api\/?$/, '')
    return `${baseUrl}${path}`
}

const fetchKaryawan = async () => {
    try {
        const { data } = await api.get(`/employees/${route.params.id}`)
        karyawan.value = data.employee
    } catch (err) {
        karyawan.value = {
            id: 'KRY003X',
            name: 'Raka Pratama',
            position: 'Frontend Developer',
            phone: '0812-3456-7890',
            division: {
                id: 'DIV005',
                name: 'Frontend'
            },
            image: null
        }
    }
}

onMounted(fetchKaryawan)
</script>