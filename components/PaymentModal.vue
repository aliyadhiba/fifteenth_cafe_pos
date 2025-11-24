<template>
  <TransitionRoot appear :show="isOpen" as="template">
    <Dialog as="div" @close="$emit('close')" class="relative z-10">
      <TransitionChild
        as="template"
        enter="duration-300 ease-out"
        enter-from="opacity-0"
        enter-to="opacity-100"
        leave="duration-200 ease-in"
        leave-from="opacity-100"
        leave-to="opacity-0"
      >
        <div class="fixed inset-0 bg-black bg-opacity-25" />
      </TransitionChild>

      <div class="fixed inset-0 overflow-y-auto">
        <div class="flex min-h-full items-center justify-center p-4 text-center">
          <TransitionChild
            as="template"
            enter="duration-300 ease-out"
            enter-from="opacity-0 scale-95"
            enter-to="opacity-100 scale-100"
            leave="duration-200 ease-in"
            leave-from="opacity-100 scale-100"
            leave-to="opacity-0 scale-95"
          >
            <DialogPanel class="w-full max-w-md transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all">
              <DialogTitle as="h3" class="text-lg font-medium leading-6 text-gray-900 mb-4">
                Payment Details
              </DialogTitle>
              
              <form @submit.prevent="processPayment">
                <!-- Customer Information -->
                <div class="mb-6">
                  <h4 class="text-sm font-medium text-gray-700 mb-3">Customer Information</h4>
                  <div class="space-y-3">
                    <div>
                      <label class="block text-sm font-medium text-gray-700 mb-1">Name</label>
                      <input
                        v-model="form.customerName"
                        type="text"
                        required
                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                        placeholder="Enter customer name"
                      />
                    </div>
                    <div>
                      <label class="block text-sm font-medium text-gray-700 mb-1">Email (Optional)</label>
                      <input
                        v-model="form.customerEmail"
                        type="email"
                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                        placeholder="Enter email address"
                      />
                    </div>
                  </div>
                </div>
                
                <!-- Payment Method -->
                <div class="mb-6">
                  <h4 class="text-sm font-medium text-gray-700 mb-3">Payment Method</h4>
                  <div class="grid grid-cols-2 gap-3">
                    <button
                      type="button"
                      @click="form.paymentMethod = 'cash'"
                      class="p-3 border rounded-lg text-center transition-colors duration-200"
                      :class="form.paymentMethod === 'cash' 
                        ? 'border-blue-500 bg-blue-50 text-blue-700' 
                        : 'border-gray-300 hover:border-gray-400'"
                    >
                      <BanknotesIcon class="w-6 h-6 mx-auto mb-1" />
                      <span class="text-sm font-medium">Cash</span>
                    </button>
                    <button
                      type="button"
                      @click="form.paymentMethod = 'card'"
                      class="p-3 border rounded-lg text-center transition-colors duration-200"
                      :class="form.paymentMethod === 'card' 
                        ? 'border-blue-500 bg-blue-50 text-blue-700' 
                        : 'border-gray-300 hover:border-gray-400'"
                    >
                      <CreditCardIcon class="w-6 h-6 mx-auto mb-1" />
                      <span class="text-sm font-medium">Card</span>
                    </button>
                  </div>
                </div>
                
                <!-- Cash Payment -->
                <div v-if="form.paymentMethod === 'cash'" class="mb-6">
                  <label class="block text-sm font-medium text-gray-700 mb-1">Amount Received</label>
                  <input
                    v-model.number="form.amountReceived"
                    type="number"
                    step="0.01"
                    min="0"
                    required
                    class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                    placeholder="0.00"
                  />
                  <div v-if="form.amountReceived >= total" class="mt-2 p-2 bg-green-50 rounded-md">
                    <p class="text-sm text-green-700">
                      Change: ${{ (form.amountReceived - total).toFixed(2) }}
                    </p>
                  </div>
                  <div v-else-if="form.amountReceived > 0" class="mt-2 p-2 bg-red-50 rounded-md">
                    <p class="text-sm text-red-700">
                      Insufficient amount. Need ${{ (total - form.amountReceived).toFixed(2) }} more.
                    </p>
                  </div>
                </div>
                
                <!-- Card Payment -->
                <div v-if="form.paymentMethod === 'card'" class="mb-6">
                  <div class="p-3 bg-blue-50 rounded-lg">
                    <p class="text-sm text-blue-700">
                      Please insert or tap card on the payment terminal.
                    </p>
                  </div>
                </div>
                
                <!-- Order Summary -->
                <div class="mb-6 p-3 bg-gray-50 rounded-lg">
                  <h4 class="text-sm font-medium text-gray-700 mb-2">Order Summary</h4>
                  <div class="text-sm space-y-1">
                    <div class="flex justify-between">
                      <span class="text-gray-600">Total Amount:</span>
                      <span class="font-medium">${{ total.toFixed(2) }}</span>
                    </div>
                  </div>
                </div>
                
                <!-- Action Buttons -->
                <div class="flex space-x-3">
                  <button
                    type="button"
                    @click="$emit('close')"
                    class="flex-1 px-4 py-2 text-sm font-medium text-gray-700 bg-gray-100 border border-gray-300 rounded-md hover:bg-gray-200 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
                  >
                    Cancel
                  </button>
                  <button
                    type="submit"
                    :disabled="!canProcessPayment"
                    class="flex-1 px-4 py-2 text-sm font-medium text-white bg-blue-600 border border-transparent rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:bg-gray-300 disabled:cursor-not-allowed"
                  >
                    Process Payment
                  </button>
                </div>
              </form>
            </DialogPanel>
          </TransitionChild>
        </div>
      </div>
    </Dialog>
  </TransitionRoot>
</template>

<script setup>
import { ref, computed } from 'vue'
import { 
  Dialog, 
  DialogPanel, 
  DialogTitle, 
  TransitionChild, 
  TransitionRoot 
} from '@headlessui/vue'
import { BanknotesIcon, CreditCardIcon } from '@heroicons/vue/24/outline'

const props = defineProps({
  isOpen: Boolean,
  total: Number,
  cartItems: Array
})

const emit = defineEmits(['close', 'payment-complete'])

const form = ref({
  customerName: '',
  customerEmail: '',
  paymentMethod: 'cash',
  amountReceived: 0
})

const canProcessPayment = computed(() => {
  if (!form.value.customerName) return false
  if (form.value.paymentMethod === 'cash') {
    return form.value.amountReceived >= props.total
  }
  return true
})

const processPayment = () => {
  if (!canProcessPayment.value) return
  
  const paymentData = {
    customerName: form.value.customerName,
    customerEmail: form.value.customerEmail,
    paymentMethod: form.value.paymentMethod,
    amountReceived: form.value.amountReceived,
    change: form.value.paymentMethod === 'cash' ? form.value.amountReceived - props.total : 0
  }
  
  emit('payment-complete', paymentData)
  
  // Reset form
  form.value = {
    customerName: '',
    customerEmail: '',
    paymentMethod: 'cash',
    amountReceived: 0
  }
}
</script>