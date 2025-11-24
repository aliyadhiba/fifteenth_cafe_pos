<template>
  <div class="flex flex-col h-screen bg-gray-50 text-gray-800">

    <!-- ===== BAGIAN UTAMA (SIDEBAR + DETAIL) ===== -->
    <div class="flex flex-1 overflow-hidden">

      <!-- ==================== SIDEBAR ==================== -->
      <div class="w-1/3 border-r border-gray-200 flex flex-col">
        <div class="p-4 relative flex items-center gap-2">
          <button 
            @click="clearOrders"
            class="bg-red-500 text-white text-sm px-3 py-2 rounded-lg hover:bg-red-600 transition"
          >
            Reset History
          </button>

          <div class="flex-1 relative">
            <MagnifyingGlassIcon class="w-5 h-5 text-gray-400 absolute left-3 top-1/2 transform -translate-y-1/2" />
            <input 
              v-model="search"
              type="text" 
              placeholder="Nomor Struk atau Invoice"
              class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400"
            />
          </div>
        </div>

        <!-- LIST HISTORY -->
        <div class="flex-1 overflow-y-auto">
          <div v-if="filteredOrders.length === 0" class="text-center text-gray-500 italic mt-10">
            Tidak ada history untuk outlet ini.
          </div>

          <div
            v-for="(order, index) in filteredOrders"
            :key="index"
            @click="selectOrder(order)"
            :class="[
              'p-4 cursor-pointer transition-all border-l-4 mb-1 rounded-r-lg',
              selectedOrder === order
                ? 'bg-blue-600 text-white border-blue-700'
                : 'hover:bg-gray-100 border-transparent'
            ]"
          >
            <div class="flex justify-between items-center">
              <p :class="selectedOrder === order ? 'font-semibold text-white' : 'font-semibold text-gray-800'">
                {{ formatCurrency(getOrderTotal(order)) }}
              </p>
              <span class="text-sm opacity-80">{{ formatTime(order.date) }}</span>
            </div>

            <p class="text-sm truncate opacity-90">
              {{ (order.items || []).map(i => i.name).join(', ') }}
            </p>
          </div>
        </div>
      </div>

      <!-- ==================== DETAIL TRANSAKSI ==================== -->
      <div class="w-2/3 flex flex-col">
        <div class="p-6 flex-1 overflow-y-auto">
          <h1 class="text-xl font-bold mb-4">History Order</h1>

          <div v-if="selectedOrder" class="space-y-6">

            <button 
              @click="openReceipt"
              class="border border-gray-400 px-4 py-2 rounded-lg hover:bg-blue-600 hover:text-white transition flex items-center gap-2"
            >
              <PaperAirplaneIcon class="w-5 h-5" />
              Kirim Struk
            </button>

            <!-- DETAIL CARD -->
            <div class="bg-white border border-gray-200 rounded-xl shadow-sm p-4">
              <h2 class="font-semibold text-gray-700 mb-3 flex items-center gap-2">
                <InformationCircleIcon class="w-5 h-5 text-blue-600" />
                Detail Transaksi
              </h2>

              <div class="space-y-3 text-sm">

                <div class="flex items-center gap-2">
                  <CreditCardIcon class="w-4 h-4 text-gray-500" />
                  <p class="text-gray-500 w-40">Metode Pembayaran</p>
                  <p class="font-medium">{{ selectedOrder.paymentMethod || '-' }}</p>
                </div>

                <div class="flex items-center gap-2">
                  <ReceiptPercentIcon class="w-4 h-4 text-gray-500" />
                  <p class="text-gray-500 w-40">Nomor Struk</p>
                  <p class="font-medium">{{ selectedOrder.orderNumber }}</p>
                </div>

                <div class="flex items-center gap-2">
                  <ClockIcon class="w-4 h-4 text-gray-500" />
                  <p class="text-gray-500 w-40">Waktu Pembelian</p>
                  <p class="font-medium">{{ formatDate(selectedOrder.date) }}</p>
                </div>

              </div>
            </div>

            <!-- PRODUK CARD -->
            <div class="bg-white border border-gray-200 rounded-xl shadow-sm p-4">
              <h2 class="font-semibold text-gray-700 mb-3 flex items-center gap-2">
                <ShoppingBagIcon class="w-5 h-5 text-blue-600" />
                Produk
              </h2>

              <ul class="divide-y divide-gray-200 border border-gray-200 rounded-lg overflow-hidden">
                <li
                  v-for="(item, i) in selectedOrder.items"
                  :key="i"
                  class="p-3 flex justify-between items-center text-sm hover:bg-gray-50 transition"
                >
                  <div class="flex items-center gap-2">
                    <ChevronRightIcon class="w-4 h-4 text-gray-400" />
                    <span>{{ item.quantity }}x {{ item.name }}</span>
                  </div>

                  <span class="font-medium">
                    {{ formatCurrency(item.price * item.quantity) }}
                  </span>
                </li>
              </ul>
            </div>

          </div>

          <div v-else class="text-gray-500 mt-10 italic text-center">
            Pilih transaksi untuk melihat detail.
          </div>
        </div>
      </div>
    </div>

    <!-- ==================== FOOTER FULL WIDTH ==================== -->
    <div class="bg-blue-700 text-white py-3 text-center font-semibold text-lg">
      Fifteenth
    </div>

  </div>

  <!-- =============== MODAL STRUK =============== -->
  <div 
    v-if="showReceiptModal" 
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-30 px-4"
  >
    <div 
      class="bg-white rounded-xl shadow-xl w-full max-w-md relative flex flex-col max-h-[90vh]"
    >
      <button 
        @click="showReceiptModal = false"
        class="absolute top-3 right-3 text-gray-500 hover:text-gray-800"
      >✕</button>

      <div class="overflow-y-auto p-6">

        <div class="text-center">
          <div class="flex justify-center mb-2">
            <div class="w-12 h-12 rounded-full bg-green-100 flex items-center justify-center">
              <CheckCircleIcon class="w-8 h-8 text-green-600" />
            </div>
          </div>
          <h2 class="text-xl font-bold text-gray-800">Payment Successful!</h2>
          <p class="text-gray-500 mt-1">POS System<br><span class="text-sm">Receipt</span></p>
        </div>

        <hr class="my-4" />

        <div class="text-sm space-y-1">
          <div class="flex justify-between"><span>Order #:</span><span>{{ r.orderNumber }}</span></div>
          <div class="flex justify-between"><span>Date:</span><span>{{ formatDate(r.date) }}</span></div>
          <div class="flex justify-between"><span>Customer:</span><span>{{ r.customer || '-' }}</span></div>
          <div class="flex justify-between"><span>Email:</span><span>{{ r.email || '-' }}</span></div>
        </div>

        <div class="mt-3 border-t pt-3">
          <p class="font-semibold mb-1">Items:</p>
          <div v-for="(item, i) in r.items" :key="i" class="flex justify-between text-sm">
            <span>{{ item.name }} x{{ item.quantity }}</span>
            <span>{{ formatCurrency(item.price * item.quantity) }}</span>
          </div>
        </div>

        <div class="mt-3 text-sm space-y-1">
          <div class="flex justify-between"><span>Subtotal:</span><span>{{ formatCurrency(r.subtotal) }}</span></div>
          <div class="flex justify-between"><span>Tax:</span><span>{{ formatCurrency(r.tax) }}</span></div>
          <div class="flex justify-between font-bold text-lg mt-2"><span>Total:</span><span>{{ formatCurrency(r.total) }}</span></div>
        </div>

        <div class="mt-4 text-sm space-y-1">
          <div class="flex justify-between"><span>Payment Method:</span><span>{{ r.paymentMethod }}</span></div>
          <div class="flex justify-between"><span>Amount Received:</span><span>{{ formatCurrency(r.amountReceived) }}</span></div>
          <div class="flex justify-between"><span>Change:</span><span>{{ formatCurrency(r.change) }}</span></div>
        </div>

        <hr class="my-4" />

        <div class="text-center text-sm text-gray-600">
          <p>Thank you for your business!</p>
          <p>Have a great day!</p>
        </div>
      </div>

      <div class="border-t p-4 flex justify-between bg-gray-50 rounded-b-xl">
        <button 
          @click="printReceipt()"
          class="border border-gray-400 text-gray-700 px-4 py-2 rounded-md hover:bg-gray-100"
        >
          Print Receipt
        </button>
        <button 
          @click="showReceiptModal = false"
          class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700"
        >
          Close
        </button>
      </div>
    </div>
  </div>

</template>

<script setup>
import { 
  InformationCircleIcon, ClockIcon, ShoppingBagIcon, PaperAirplaneIcon,
  CreditCardIcon, ChevronRightIcon, ReceiptPercentIcon, MagnifyingGlassIcon, CheckCircleIcon
} from '@heroicons/vue/24/outline'

import { ref, computed, onMounted } from 'vue'

const orders = ref([])
const selectedOrder = ref(null)
const search = ref('')
const showReceiptModal = ref(false)
const selectedOutlet = ref(null)
const r = ref({})

// LOAD DATA
onMounted(() => {
  const outletData = localStorage.getItem('selectedOutlet')
  if (outletData) selectedOutlet.value = JSON.parse(outletData)

  const saved = localStorage.getItem('orders')
  let allOrders = []
  try { allOrders = saved ? JSON.parse(saved) : [] } catch { allOrders = [] }

  if (selectedOutlet.value) {
    orders.value = allOrders.filter(o => o.outletId === selectedOutlet.value.id)
  }
})

// FILTER LIST
const filteredOrders = computed(() => {
  let list = orders.value.slice().sort((a, b) => new Date(b.date) - new Date(a.date))
  if (!search.value) return list
  return list.filter(o => (o.orderNumber || '').toLowerCase().includes(search.value.toLowerCase()))
})

// CLEAR HISTORY
const clearOrders = () => {
  if (!selectedOutlet.value) return alert('Outlet belum dipilih.')

  if (confirm(`Yakin ingin menghapus semua history untuk ${selectedOutlet.value.name}?`)) {
    const allOrders = JSON.parse(localStorage.getItem('orders') || '[]')
    const updated = allOrders.filter(o => o.outletId !== selectedOutlet.value.id)
    localStorage.setItem('orders', JSON.stringify(updated))

    orders.value = []
    selectedOrder.value = null
  }
}

const selectOrder = (order) => selectedOrder.value = order

// FORMATTING
const toDate = (d) => {
  const t = new Date(d)
  return isNaN(t.getTime()) ? new Date() : t
}

const formatDate = (d) =>
  toDate(d).toLocaleString('id-ID', { dateStyle: 'long', timeStyle: 'short' })

const formatTime = (d) =>
  toDate(d).toLocaleString('id-ID', {
    day: '2-digit',
    month: 'short',
    hour: '2-digit',
    minute: '2-digit'
  })

const formatCurrency = (n) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(n || 0)

// HITUNG TOTAL
const computeTotals = (order) => {
  const items = order.items || []
  const subtotal = items.reduce((t, i) => t + (i.price * i.quantity), 0)
  const tax = order.tax || subtotal * 0.08
  const total = order.total || subtotal + tax
  return { subtotal, tax, total }
}

const getOrderTotal = (order) => order.total || computeTotals(order).total

// STRUK
const openReceipt = () => {
  const o = selectedOrder.value
  if (!o) return
  r.value = { ...o, ...computeTotals(o) }
  showReceiptModal.value = true
}

const printReceipt = () => {
  const html = `
  <html><head><title>Receipt ${r.value.orderNumber}</title>
  <style>
  body { font-family: Arial; padding: 10px; }
  .line { border-top: 1px dashed #aaa; margin: 8px 0; }
  .row { display: flex; justify-content: space-between; font-size: 13px; }
  .total { font-weight: bold; font-size: 14px; }
  </style>
  </head><body>

  <h2>${selectedOutlet.value?.name || 'Outlet'}</h2>
  <div class="line"></div>

  <div class="row"><span>Order #</span><span>${r.value.orderNumber}</span></div>
  <div class="row"><span>Date</span><span>${formatDate(r.value.date)}</span></div>

  ${r.value.items.map(i => `
    <div class="row"><span>${i.quantity}x ${i.name}</span><span>${formatCurrency(i.price * i.quantity)}</span></div>
  `).join('')}

  <div class="line"></div>
  <div class="row total"><span>Total</span><span>${formatCurrency(r.value.total)}</span></div>

  <p style="margin-top: 10px;">Terima kasih!</p>

  </body></html>
  `

  const w = window.open('', '', 'width=400,height=600')
  w.document.write(html)
  w.document.close()
  setTimeout(() => w.print(), 300)
}
</script>
