<template>
  <div class="flex h-screen bg-white font-sans">
    <!-- Sidebar -->
    <div class="w-1/4 bg-gray-50 border-r flex flex-col">
      <!-- Header Sidebar -->
      <div class="p-6 flex flex-col items-center border-b">
        <div class="bg-gray-300 rounded-lg w-16 h-16 flex items-center justify-center mb-3">
          <span class="text-gray-700 text-2xl font-bold">Rp</span>
        </div>
        <p class="text-gray-700 font-semibold text-lg">Shift</p>
      </div>

      <!-- Menu Navigasi -->
      <nav class="flex-1 px-4 mt-4 space-y-1 text-gray-700">
        <div class="w-full text-left px-4 py-2 bg-gray-100 text-gray-700 font-medium rounded-r-lg cursor-default select-none">
          Pengaturan Shift
        </div>
        <div class="w-full text-left px-4 py-2 bg-gray-100 text-gray-700 font-medium rounded-r-lg cursor-default select-none">
          Pilihan Shift
        </div>
        <button
          @click="setActive('current')"
          :class="activeTab === 'current' ? activeButton : normalButton"
        >
          Shift Saat Ini
        </button>
        <button
          @click="setActive('history')"
          :class="activeTab === 'history' ? activeButton : normalButton"
        >
          History Shift
        </button>
      </nav>

      <!-- Info Outlet -->
      <div class="border-t py-3 text-center text-sm text-gray-500">
        {{ currentOutlet || 'Outlet Tidak Dikenali' }}
      </div>
    </div>

    <!-- Konten kanan -->
    <div class="flex-1 p-8 bg-white overflow-y-auto">
      <!-- Shift Saat Ini -->
      <div v-if="activeTab === 'current'">
        <div class="flex justify-between items-center mb-6">
          <h1 class="text-2xl font-semibold text-gray-800">Shift Saat Ini</h1>
          <div class="flex gap-3">
            <button
              @click="endShift"
              class="bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition"
            >
              Akhiri Shift
            </button>
            <button
              @click="printReport"
              class="border border-gray-400 px-4 py-2 rounded-lg hover:bg-gray-100 transition"
            >
              Cetak Laporan Shift
            </button>
          </div>
        </div>

        <div v-if="!currentShift">
          <p class="text-gray-500 italic">Belum ada shift yang dimulai.</p>
          <button
            @click="startShift"
            class="mt-4 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700"
          >
            Mulai Shift
          </button>
        </div>

        <div v-else class="bg-gray-50 border rounded-lg p-6 space-y-4">
          <h2 class="text-lg font-semibold text-gray-700 border-b pb-2">Detail Shift</h2>
          <div class="grid grid-cols-2 gap-y-3">
            <p class="text-gray-600">Nama</p>
            <p class="text-right text-gray-800 font-medium">{{ currentShift.kasir }}</p>

            <p class="text-gray-600">Outlet</p>
            <p class="text-right text-gray-800 font-medium">{{ currentShift.outlet }}</p>

            <p class="text-gray-600">Mulai Shift</p>
            <p class="text-right text-gray-800 font-medium">{{ formatDate(currentShift.start) }}</p>

            <p class="text-gray-600">Status</p>
            <p class="text-right text-gray-800 font-medium">{{ currentShift.status }}</p>
          </div>

          <div class="border-t pt-4 text-sm text-gray-500">
            <p>Detail Pemesanan</p>
            <p class="mt-2">Produk Terjual: <strong>{{ soldProducts }}</strong></p>
          </div>
        </div>
      </div>

      <!-- Histori Shift -->
      <div v-else-if="activeTab === 'history'">
        <h1 class="text-2xl font-semibold text-gray-800 mb-4">History Shift</h1>
        <div v-if="!filteredShiftHistory.length" class="text-gray-500 italic">
  Belum ada history shift untuk kasir ini.
</div>
        <ul>
          <li
  v-for="(s, i) in filteredShiftHistory"
  :key="i"
  class="p-3 border-b border-gray-200 hover:bg-gray-50 flex justify-between"
>
  <span>{{ formatDate(s.start) }} - {{ formatDate(s.end) }}</span>
  <span class="text-sm text-gray-500">{{ s.kasir }}</span>
</li>

        </ul>
        <div class="mt-4 text-right">
          <button
            @click="resetHistory"
            class="bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600"
          >
            Reset History
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- FOOTER BAR -->
    <div class="footer-bar">Fifteenth</div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'

const activeTab = ref('current')
const currentUser = ref(localStorage.getItem('currentUser') || 'Kasir')
const currentOutlet = ref(localStorage.getItem('currentOutlet') || 'Outlet Belum Dipilih')
const currentShift = ref(null)
const shiftHistory = ref([])
const transactions = ref([]) // reactive transactions

const activeButton = 'w-full text-left px-4 py-2 bg-blue-600 text-white rounded-r-lg'
const normalButton = 'w-full text-left px-4 py-2 hover:bg-blue-100 rounded-r-lg transition'

const setActive = (tab) => activeTab.value = tab

const filteredShiftHistory = computed(() => {
  return shiftHistory.value.filter(s => s.kasir === currentUser.value)
})


// Load data awal
const loadData = () => {
  const outletKey = currentOutlet.value.replace(/\s+/g,'_').toLowerCase()
  const savedCurrent = localStorage.getItem(`currentShift_${outletKey}`)
  const savedHistory = localStorage.getItem(`shiftHistory_${outletKey}`)
  const savedTrans = localStorage.getItem(`transactions_${outletKey}`)
  currentShift.value = savedCurrent ? JSON.parse(savedCurrent) : null
  shiftHistory.value = savedHistory ? JSON.parse(savedHistory) : []
  transactions.value = savedTrans ? JSON.parse(savedTrans) : []
}

onMounted(() => {
  loadData()
})

// Watch outlet
watch(currentOutlet, loadData)

// FORMAT DATE
const formatDate = d => {
  if(!d) return '-'
  return new Date(d).toLocaleString('id-ID', { weekday:'long', day:'2-digit', month:'long', year:'numeric', hour:'2-digit', minute:'2-digit' })
}

// SHIFT
const startShift = () => {
  const outletKey = currentOutlet.value.replace(/\s+/g,'_').toLowerCase()
  // === PERUBAHAN: Reset transaksi saat mulai shift baru ===
  transactions.value = [] // Reset array transaksi ke kosong
  localStorage.setItem(`transactions_${outletKey}`, JSON.stringify(transactions.value)) // Simpan reset ke localStorage
  // Mulai shift baru
  currentShift.value = { kasir: currentUser.value, outlet: currentOutlet.value, start:new Date().toISOString(), status:'Aktif' }
  localStorage.setItem(`currentShift_${outletKey}`, JSON.stringify(currentShift.value))
}

const endShift = () => {
  if (!currentShift.value) return

  const outletKey = currentOutlet.value.replace(/\s+/g, '_').toLowerCase()

  const ended = { 
    ...currentShift.value, 
    end: new Date().toISOString(), 
    status: 'Selesai' 
  }

  shiftHistory.value.push(ended)
  localStorage.setItem(`shiftHistory_${outletKey}`, JSON.stringify(shiftHistory.value))

  // WAJIB: hapus shift aktif dari localStorage
  localStorage.removeItem(`currentShift_${outletKey}`)

  currentShift.value = null
}


// PRODUK TERJUAL / REFUND
const soldProducts = computed(() => {
  if(!currentShift.value) return 0
  return transactions.value
    .filter(t => t.shiftId === currentShift.value.start && !t.refund)
    .reduce((sum,t)=> sum+t.qty,0)
})

const refundedProducts = computed(() => {
  if(!currentShift.value) return 0
  return transactions.value
    .filter(t => t.shiftId === currentShift.value.start && t.refund)
    .reduce((sum,t)=> sum+t.qty,0)
})

// Fungsi untuk POS menambahkan transaksi
const addPOSTransaction = (product, qty, refund=false) => {
  if (!currentShift.value) return alert("Mulai shift dulu!")
  const outletKey = currentOutlet.value.replace(/\s+/g,'_').toLowerCase()
  transactions.value.push({
    product,
    qty,
    refund,
    shiftId: currentShift.value.start,
    timestamp: new Date().toISOString()
  })
  localStorage.setItem(`transactions_${outletKey}`, JSON.stringify(transactions.value))
}

// PRINT REPORT
const printReport = () => {
  if(!currentShift.value) return
  const report = `
SHIFT REPORT
Kasir: ${currentShift.value.kasir}
Outlet: ${currentShift.value.outlet}
Mulai: ${new Date(currentShift.value.start).toLocaleString('id-ID')}
Status: ${currentShift.value.status}
Produk Terjual: ${soldProducts.value}
Produk Refund: ${refundedProducts.value}
`
  const w = window.open('','width=400,height=600')
  w.document.write(`<pre>${report}</pre>`)
  w.print()
}

// RESET HISTORY
const resetHistory = () => {
  if(confirm('Yakin hapus semua history dan transaksi?')){
    const outletKey = currentOutlet.value.replace(/\s+/g,'_').toLowerCase()
    shiftHistory.value = []
    transactions.value = []
    localStorage.removeItem(`shiftHistory_${outletKey}`)
    localStorage.removeItem(`transactions_${outletKey}`)
  }
}
</script>

<style scoped>
button { transition: 0.3s; }

.footer-bar {
  height: 60px;
  background: #175cd3;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 18px;
  position: fixed;
  bottom: 0;
  width: 100%;
}
</style>