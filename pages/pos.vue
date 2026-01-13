<template>
  <div class="min-h-screen flex flex-col md:flex-row bg-gray-100">

    <!-- ================= PRODUCTS ================= -->
    <div class="flex-1 p-4 md:p-6">
      <h2 class="text-2xl font-bold mb-4">Cashier</h2>

      <!-- Search -->
      <div class="relative mb-4">
        <MagnifyingGlassIcon
          class="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400"
        />
        <input
          v-model="searchQuery"
          placeholder="Search products..."
          class="w-full pl-10 pr-4 py-2 border rounded-lg"
        />
      </div>

      <!-- Categories -->
      <div class="flex gap-2 overflow-x-auto mb-4">
        <button
          v-for="c in categories"
          :key="c"
          @click="selectedCategory = c"
          class="px-4 py-2 rounded-lg whitespace-nowrap"
          :class="selectedCategory === c ? 'bg-blue-600 text-white' : 'bg-gray-200'"
        >
          {{ c }}
        </button>
      </div>

      <!-- Products -->
      <div
        class="flex gap-4 overflow-x-auto md:grid md:grid-cols-3 lg:grid-cols-4 md:overflow-visible"
      >
        <div
          v-for="p in filteredProducts"
          :key="p.id"
          @click="addToCart(p)"
          class="min-w-[160px] md:min-w-0 bg-white p-4 rounded-lg shadow cursor-pointer"
        >
          <img
            :src="p.image"
            class="w-full h-32 object-cover rounded mb-2"
          />
          <h3 class="font-semibold">{{ p.name }}</h3>
          <p class="text-blue-600 font-bold">${{ p.price }}</p>
        </div>
      </div>
    </div>

    <!-- ================= CART ================= -->
    <div class="w-full md:w-96 bg-white p-4 md:p-6 shadow-lg overflow-y-auto">

      <h3 class="text-xl font-bold mb-4">Current Order</h3>

      <!-- Cart items -->
      <div class="space-y-3 max-h-72 overflow-y-auto mb-4">
        <div
          v-for="item in cartItems"
          :key="item.id"
          class="flex justify-between items-center bg-gray-100 p-3 rounded"
        >
          <div>
            <p class="font-medium">{{ item.name }}</p>
            <p class="text-sm">${{ item.price }} x {{ item.quantity }}</p>
          </div>
          <div class="flex items-center gap-2">
            <button @click="updateQuantity(item.id, item.quantity - 1)">
              <MinusIcon class="w-4 h-4" />
            </button>
            <span>{{ item.quantity }}</span>
            <button @click="updateQuantity(item.id, item.quantity + 1)">
              <PlusIcon class="w-4 h-4" />
            </button>
            <button @click="removeFromCart(item.id)">
              <TrashIcon class="w-4 h-4 text-red-500" />
            </button>
          </div>
        </div>
      </div>

      <!-- Summary -->
      <div class="border-t pt-4 pb-24 md:pb-4">
        <div class="flex justify-between mb-2">
          <span>Subtotal</span>
          <span>${{ subtotal.toFixed(2) }}</span>
        </div>
        <div class="flex justify-between mb-2">
          <span>Tax</span>
          <span>${{ tax.toFixed(2) }}</span>
        </div>
        <div class="flex justify-between font-bold text-lg mb-4">
          <span>Total</span>
          <span>${{ total.toFixed(2) }}</span>
        </div>

        <button
          @click="openPaymentModal"
          :disabled="cartItems.length === 0"
          class="w-full bg-blue-600 text-white py-3 rounded-lg mb-2 disabled:bg-gray-300"
        >
          Process Payment
        </button>

        <button
          @click="clearCart"
          class="w-full bg-gray-200 py-2 rounded-lg"
        >
          Clear Order
        </button>
      </div>
    </div>

    <!-- ================= MODALS ================= -->
    <PaymentModal
  v-if="showPaymentModal"
  :is-open="showPaymentModal"
  :total="total"
  :cart-items="cartItems"
  @close="closePaymentModal"
  @payment-complete="handlePaymentComplete"
/>

<ReceiptModal
  v-if="showReceiptModal"
  :is-open="showReceiptModal"
  :order-data="orderData"
  @close="showReceiptModal = false"
/>


    <!-- ================= FOOTER ================= -->
    <div
      class="w-full bg-blue-600 text-white text-center py-3 font-semibold md:fixed md:bottom-0"
    >
      Fifteenth
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { navigateTo } from '#app'
import {
  MagnifyingGlassIcon,
  MinusIcon,
  PlusIcon,
  TrashIcon
} from '@heroicons/vue/24/outline'

import PaymentModal from '~/components/PaymentModal.vue'
import ReceiptModal from '~/components/ReceiptModal.vue'

/* ================= AUTH GUARD ================= */
onMounted(() => {
  const isLoggedIn = localStorage.getItem('isLoggedIn')
  const outlet = localStorage.getItem('selectedOutlet')

  if (isLoggedIn !== 'true') {
    navigateTo('/login')
    return
  }

  if (!outlet) {
    navigateTo('/outlet')
    return
  }
})


/* ================= STATE ================= */
const searchQuery = ref('')
const selectedCategory = ref('All')
const cartItems = ref([])
const showPaymentModal = ref(false)
const showReceiptModal = ref(false)
const orderData = ref(null)

/* ================= DATA ================= */
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

/* ================= COMPUTED ================= */
const filteredProducts = computed(() =>
  products.filter(p =>
    (selectedCategory.value === 'All' || p.category === selectedCategory.value) &&
    p.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
)

const subtotal = computed(() =>
  cartItems.value.reduce((sum, i) => sum + i.price * i.quantity, 0)
)

const tax = computed(() => subtotal.value * 0.08)
const total = computed(() => subtotal.value + tax.value)

/* ================= CART ================= */
const addToCart = product => {
  const item = cartItems.value.find(i => i.id === product.id)
  item ? item.quantity++ : cartItems.value.push({ ...product, quantity: 1 })
}

const updateQuantity = (id, qty) => {
  const item = cartItems.value.find(i => i.id === id)
  if (!item) return
  qty <= 0 ? removeFromCart(id) : item.quantity = qty
}

const removeFromCart = id => {
  cartItems.value = cartItems.value.filter(i => i.id !== id)
}

const clearCart = () => {
  cartItems.value = []
}

/* ================= PAYMENT ================= */
const openPaymentModal = () => showPaymentModal.value = true
const closePaymentModal = () => showPaymentModal.value = false

const handlePaymentComplete = (data) => {
  const outlet = JSON.parse(localStorage.getItem('selectedOutlet'))
  if (!outlet) return alert('Outlet belum dipilih')

  const outletKey = outlet.name.replace(/\s+/g, '_').toLowerCase()
  const currentShift = JSON.parse(
    localStorage.getItem(`currentShift_${outletKey}`)
  )

  if (!currentShift) {
    alert('Shift belum dimulai')
    return
  }

  // ================= ORDER OBJECT =================
  const order = {
    orderNumber: `ORD-${Date.now()}`,
    date: new Date().toISOString(),
    items: [...cartItems.value],
    subtotal: subtotal.value,
    tax: tax.value,
    total: total.value,
    paymentMethod: data.paymentMethod,
    amountReceived: data.amountReceived,
    change: data.change, 
    customerName: data.customerName || 'Guest',
    outletId: outlet.id
  }

  // ================= SAVE HISTORY =================
  const orders = JSON.parse(localStorage.getItem('orders') || '[]')
  orders.push(order)
  localStorage.setItem('orders', JSON.stringify(orders))

  // ================= SAVE ACTIVITY =================
  const transactions =
    JSON.parse(localStorage.getItem(`transactions_${outletKey}`) || '[]')

  cartItems.value.forEach(item => {
    transactions.push({
      product: item.name,
      qty: item.quantity,
      shiftId: currentShift.start,
      refund: false
    })
  })

  localStorage.setItem(
    `transactions_${outletKey}`,
    JSON.stringify(transactions)
  )

  // ================= UI =================
  orderData.value = order
  clearCart()
  closePaymentModal()
  showReceiptModal.value = true
}
</script>