<template>
  <div class="flex h-full">
    <!-- Products Section -->
    <div class="flex-1 p-6">
      <div class="mb-6">
        <h2 class="text-2xl font-bold text-gray-800 mb-4">Cashier</h2>
        
        <!-- Search Bar -->
        <div class="relative">
          <MagnifyingGlassIcon class="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400" />
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Search products..."
            class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </div>
      </div>
      
      <!-- Categories -->
      <div class="mb-6">
        <div class="flex space-x-2 overflow-x-auto">
          <button
            v-for="category in categories"
            :key="category"
            @click="selectedCategory = category"
            class="px-4 py-2 rounded-lg whitespace-nowrap transition-colors duration-200"
            :class="selectedCategory === category 
              ? 'bg-blue-600 text-white' 
              : 'bg-gray-200 text-gray-700 hover:bg-gray-300'"
          >
            {{ category }}
          </button>
        </div>
      </div>
      
      <!-- Products Grid -->
      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        <div
          v-for="product in filteredProducts"
          :key="product.id"
          @click="addToCart(product)"
          class="bg-white rounded-lg shadow-md p-4 cursor-pointer hover:shadow-lg transition-shadow duration-200"
        >
          <img
            :src="product.image"
            :alt="product.name"
            class="w-full h-32 object-cover rounded-lg mb-3"
          />
          <h3 class="font-semibold text-gray-800 mb-1">{{ product.name }}</h3>
          <p class="text-lg font-bold text-blue-600">${{ product.price.toFixed(2) }}</p>
        </div>
      </div>
    </div>
    
    <!-- Cart Section -->
    <div class="w-96 bg-white shadow-lg p-6">
      <h3 class="text-xl font-bold text-gray-800 mb-4">Current Order</h3>
      
      <!-- Cart Items -->
      <div class="space-y-3 mb-6 max-h-96 overflow-y-auto">
        <div
          v-for="item in cartItems"
          :key="item.id"
          class="flex items-center justify-between p-3 bg-gray-50 rounded-lg"
        >
          <div class="flex-1">
            <h4 class="font-medium text-gray-800">{{ item.name }}</h4>
            <p class="text-sm text-gray-600">${{ item.price.toFixed(2) }} each</p>
          </div>
          <div class="flex items-center space-x-2">
            <button
              @click="updateQuantity(item.id, item.quantity - 1)"
              class="w-8 h-8 rounded-full bg-gray-200 flex items-center justify-center hover:bg-gray-300"
            >
              <MinusIcon class="w-4 h-4" />
            </button>
            <span class="w-8 text-center font-medium">{{ item.quantity }}</span>
            <button
              @click="updateQuantity(item.id, item.quantity + 1)"
              class="w-8 h-8 rounded-full bg-gray-200 flex items-center justify-center hover:bg-gray-300"
            >
              <PlusIcon class="w-4 h-4" />
            </button>
          </div>
          <button
            @click="removeFromCart(item.id)"
            class="ml-2 p-1 text-red-500 hover:text-red-700"
          >
            <TrashIcon class="w-4 h-4" />
          </button>
        </div>
      </div>
      
      <!-- Order Summary -->
      <div class="border-t pt-4">
        <div class="space-y-2 mb-4">
          <div class="flex justify-between">
            <span class="text-gray-600">Subtotal:</span>
            <span class="font-medium">${{ subtotal.toFixed(2) }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-600">Tax (8%):</span>
            <span class="font-medium">${{ tax.toFixed(2) }}</span>
          </div>
          <div class="flex justify-between text-lg font-bold">
            <span>Total:</span>
            <span>${{ total.toFixed(2) }}</span>
          </div>
        </div>
        
        <!-- Action Buttons -->
        <div class="space-y-2">
          <button
            @click="openPaymentModal"
            :disabled="cartItems.length === 0"
            class="w-full bg-blue-600 text-white py-3 rounded-lg font-medium hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed transition-colors duration-200"
          >
            Process Payment
          </button>
          <button
            @click="clearCart"
            class="w-full bg-gray-200 text-gray-700 py-2 rounded-lg font-medium hover:bg-gray-300 transition-colors duration-200"
          >
            Clear Order
          </button>
        </div>
      </div>
    </div>
    
    <!-- Payment Modal -->
    <PaymentModal
      :is-open="showPaymentModal"
      :total="total"
      :cart-items="cartItems"
      :paid="paid"
      :change="change" 
      @close="closePaymentModal"
      @payment-complete="handlePaymentComplete"
    />
    
        <ReceiptModal
      :is-open="showReceiptModal"
      :order-data="orderData"
      @close="closeReceiptModal"
    />
  </div>

  <!-- Footer Fifteenth -->
<div class="w-full bg-blue-600 text-white text-center py-3 fixed bottom-0 left-0 font-semibold text-lg">
  Fifteenth
</div>
</template>


<script setup>
import { ref, computed } from 'vue'
import { 
  MagnifyingGlassIcon,
  MinusIcon,
  PlusIcon,
  TrashIcon
} from '@heroicons/vue/24/outline'
import PaymentModal from '~/components/PaymentModal.vue'
import ReceiptModal from '~/components/ReceiptModal.vue'

// ====== STATE ======
const searchQuery = ref('')
const selectedCategory = ref('All')
const cartItems = ref([])
const showPaymentModal = ref(false)
const showReceiptModal = ref(false)
const orderData = ref(null)

// ====== DATA DUMMY PRODUK ======
const categories = ['All', 'Food', 'Beverages', 'Snacks', 'Desserts']

const products = [
  { id: 1, name: 'Burger', price: 12.99, category: 'Food', image: 'https://images.pexels.com/photos/1639557/pexels-photo-1639557.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 2, name: 'Pizza', price: 18.99, category: 'Food', image: 'https://images.pexels.com/photos/315755/pexels-photo-315755.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 3, name: 'Coffee', price: 4.99, category: 'Beverages', image: 'https://images.pexels.com/photos/302899/pexels-photo-302899.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 4, name: 'Soda', price: 2.99, category: 'Beverages', image: 'https://images.pexels.com/photos/50593/coca-cola-cold-drink-soft-drink-coke-50593.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 5, name: 'Chips', price: 3.99, category: 'Snacks', image: 'https://images.pexels.com/photos/209206/pexels-photo-209206.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 6, name: 'Ice Cream', price: 6.99, category: 'Desserts', image: 'https://images.pexels.com/photos/1362534/pexels-photo-1362534.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 7, name: 'Sandwich', price: 8.99, category: 'Food', image: 'https://images.pexels.com/photos/1600727/pexels-photo-1600727.jpeg?auto=compress&cs=tinysrgb&w=400' },
  { id: 8, name: 'Juice', price: 5.99, category: 'Beverages', image: 'https://images.pexels.com/photos/96974/pexels-photo-96974.jpeg?auto=compress&cs=tinysrgb&w=400' }
]

// ====== COMPUTED ======
const filteredProducts = computed(() => {
  let filtered = products
  if (selectedCategory.value !== 'All') {
    filtered = filtered.filter(p => p.category === selectedCategory.value)
  }
  if (searchQuery.value) {
    filtered = filtered.filter(p => p.name.toLowerCase().includes(searchQuery.value.toLowerCase()))
  }
  return filtered
})

const subtotal = computed(() => cartItems.value.reduce((sum, i) => sum + (i.price * i.quantity), 0))
const tax = computed(() => subtotal.value * 0.08)
const total = computed(() => subtotal.value + tax.value)

// ====== CART LOGIC ======
const addToCart = (product) => {
  const existing = cartItems.value.find(i => i.id === product.id)
  if (existing) existing.quantity++
  else cartItems.value.push({ ...product, quantity: 1 })
}

const updateQuantity = (id, qty) => {
  if (qty <= 0) removeFromCart(id)
  else {
    const item = cartItems.value.find(i => i.id === id)
    if (item) item.quantity = qty
  }
}

const removeFromCart = (id) => {
  cartItems.value = cartItems.value.filter(i => i.id !== id)
}

const clearCart = () => cartItems.value = []

// ====== PAYMENT ======
const openPaymentModal = () => showPaymentModal.value = true
const closePaymentModal = () => showPaymentModal.value = false

const handlePaymentComplete = (paymentData) => {
  const currentUser = localStorage.getItem('currentUser') || 'Kasir'
  const selectedOutlet = JSON.parse(localStorage.getItem('selectedOutlet') || '{}')

  const newOrder = {
    ...paymentData,
    items: [...cartItems.value],
    subtotal: subtotal.value,
    tax: tax.value,
    total: total.value,
    orderNumber: generateOrderNumber(),
    date: new Date().toISOString(),
    kasir: currentUser,
    outletId: selectedOutlet.id || null,
    outletName: selectedOutlet.name || 'Unknown Outlet'
  }

  // Simpan order seperti sebelumnya
  const savedOrders = JSON.parse(localStorage.getItem('orders') || '[]')
  savedOrders.push(newOrder)
  localStorage.setItem('orders', JSON.stringify(savedOrders))

  // === PERUBAHAN: Sinkronkan transaksi ke activity.vue ===
  // Ambil shift aktif dari localStorage (sesuai dengan activity.vue)
  const outletKey = selectedOutlet.name ? selectedOutlet.name.replace(/\s+/g, '_').toLowerCase() : 'unknown_outlet'
  const currentShift = JSON.parse(localStorage.getItem(`currentShift_${outletKey}`) || 'null')

  if (currentShift && currentShift.status === 'Aktif') {
    // Ambil transaksi yang ada
    const savedTrans = JSON.parse(localStorage.getItem(`transactions_${outletKey}`) || '[]')

    // Tambahkan setiap item di cart sebagai transaksi (produk terjual)
    cartItems.value.forEach(item => {
      savedTrans.push({
        product: item.name,  // Nama produk
        qty: item.quantity,  // Jumlah yang dibeli
        refund: false,       // Asumsikan bukan refund
        shiftId: currentShift.start,  // ID shift dari activity.vue
        timestamp: new Date().toISOString()  // Waktu transaksi
      })
    })

    // Simpan kembali ke localStorage
    localStorage.setItem(`transactions_${outletKey}`, JSON.stringify(savedTrans))
  } else {
    // Jika tidak ada shift aktif, beri peringatan (opsional)
    console.warn('Tidak ada shift aktif. Transaksi tidak dicatat untuk activity.vue.')
  }

  // Lanjutkan seperti sebelumnya
  orderData.value = newOrder
  clearCart()
  closePaymentModal()
  showReceiptModal.value = true
}

const closeReceiptModal = () => {
  showReceiptModal.value = false
  orderData.value = null
}

const generateOrderNumber = () => 'ORD-' + Date.now().toString().slice(-6)
</script>