<template>
  <div class="outlet-page">
    <div class="outlet-box">
      <h2 class="title">Pilih Outlet</h2>
      <p class="subtitle">Silakan pilih outlet tempat Anda bekerja</p>

      <div class="outlet-list">
        <div
          v-for="outlet in outlets"
          :key="outlet.id"
          @click="selectOutlet(outlet)"
          :class="[
            'p-4 cursor-pointer transition-all border-l-4 rounded-r-lg shadow-sm',
            selectedOutlet?.id === outlet.id
              ? 'bg-blue-600 text-white border-blue-700'
              : 'bg-white hover:bg-gray-50 border-transparent text-gray-800'
          ]"
        >
          <h3 class="font-semibold text-lg">{{ outlet.name }}</h3>
          <p class="text-sm opacity-80">{{ outlet.location }}</p>
        </div>
      </div>

      <transition name="fade">
        <button
          v-if="selectedOutlet"
          class="btn-continue"
          @click="goToPOS"
        >
          Lanjut ke POS
        </button>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { navigateTo } from '#app'

// 🔹 Hilangkan sidebar
definePageMeta({
  layout: false
})

// 🔸 Daftar outlet tanpa duplikat
const outlets = ref([
  { id: 1, name: 'Fifteenth Cafe by Mark Design', location: 'Jl. Lombok No. 15' },
  { id: 2, name: 'Fifteenth Coffee Kota Lama', location: 'Jl. Miwis No. 9' },
  { id: 3, name: 'Fifteenth Coffee Taman Pinang', location: 'Kabupaten Sidoarjo, Jawa Timur' },
  { id: 4, name: 'Scrt by Fifteenth', location: 'Jl. Ngagel Jaya Barat No. 63' },
])

const selectedOutlet = ref(null)

const selectOutlet = (outlet) => {
  selectedOutlet.value = outlet

  // Simpan outlet di localStorage
  localStorage.setItem('selectedOutlet', JSON.stringify(outlet))
  localStorage.setItem('currentOutlet', outlet.name)

  // === Otomatis akhiri shift aktif jika kasir berbeda ===
  const currentUser = localStorage.getItem('currentUser') || 'Kasir'
  const outletKey = outlet.name.replace(/\s+/g, '_').toLowerCase()

  const savedShift = localStorage.getItem(`currentShift_${outletKey}`)
  if (savedShift) {
    const currentShift = JSON.parse(savedShift)
    if (currentShift.status === 'Aktif' && currentShift.kasir !== currentUser) {
      // Akhiri shift lama dari kasir lain
      const endedShift = { ...currentShift, end: new Date().toISOString(), status: 'Selesai' }
      const savedHistory = JSON.parse(localStorage.getItem(`shiftHistory_${outletKey}`) || '[]')
      savedHistory.push(endedShift)
      localStorage.setItem(`shiftHistory_${outletKey}`, JSON.stringify(savedHistory))
      localStorage.removeItem(`currentShift_${outletKey}`)
      localStorage.setItem(`transactions_${outletKey}`, JSON.stringify([]))
    }
  }
}

const goToPOS = () => {
  navigateTo('/pos')
}
</script>

<style scoped>
.outlet-page {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(135deg, #f0f4f8, #e4ebf5);
  padding: 1rem;
}

.outlet-box {
  background: white;
  padding: 2rem 3rem;
  border-radius: 16px;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.08);
  text-align: center;
}

.title {
  font-size: 1.6rem;
  font-weight: 700;
  color: #1e293b;
}

.subtitle {
  margin-top: 0.25rem;
  color: #64748b;
  margin-bottom: 1.5rem;
}

.outlet-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  max-height: 300px;
  overflow-y: auto;
  padding-right: 0.25rem;
  scrollbar-width: thin;
}

/* Scrollbar */
.outlet-list::-webkit-scrollbar {
  width: 6px;
}
.outlet-list::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 10px;
}

/* Tombol lanjut */
.btn-continue {
  margin-top: 1.5rem;
  background: linear-gradient(135deg, #2563eb, #3b82f6);
  color: white;
  border: none;
  padding: 0.75rem 1rem;
  border-radius: 10px;
  cursor: pointer;
  width: 100%;
  font-weight: 600;
  letter-spacing: 0.5px;
  box-shadow: 0 4px 10px rgba(37, 99, 235, 0.3);
  transition: all 0.25s ease-in-out;
}

.btn-continue:hover {
  background: linear-gradient(135deg, #1d4ed8, #2563eb);
  transform: translateY(-2px);
}

/* Animasi tombol fade */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
