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
            <DialogPanel
              class="w-full max-w-md transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all"
            >
              <div class="text-center mb-6">
                <CheckCircleIcon class="w-16 h-16 text-green-500 mx-auto mb-4" />
                <DialogTitle as="h3" class="text-xl font-semibold text-gray-900">
                  Payment Successful!
                </DialogTitle>
              </div>

              <!-- RECEIPT TEMPLATE -->
              <div
                v-if="orderData"
                class="receipt-content"
                style="max-width: 58mm; margin: auto; font-family: 'Arial', sans-serif;"
              >
                <!-- HEADER -->
                <div style="text-align:center; margin-bottom:10px;">
                  <h2 style="font-size:16px; font-weight:bold; margin:0;">
                    {{ outlet.name }}
                  </h2>

                  <p style="font-size:11px; margin:2px 0;">
                    {{ outlet.address }}
                  </p>

                  <p style="font-size:11px; margin:2px 0;">
                    Telp: {{ outlet.phone }}
                  </p>

                  <p style="font-size:11px; margin:2px 0;">
                    WiFi: {{ outlet.wifi }}
                  </p>

                  <!-- KASIR -->
                  <p style="font-size:11px; margin:2px 0;">
                    Kasir: {{ cashierName }}
                  </p>

                  <div style="border-top:1px dashed #000; margin-top:8px;"></div>
                </div>

                <!-- ORDER INFO -->
                <div style="font-size:12px; margin-bottom:8px;">
                  <div style="display:flex; justify-content:space-between;">
                    <span>Order ID:</span> <span>{{ orderData.orderNumber }}</span>
                  </div>
                  <div style="display:flex; justify-content:space-between;">
                    <span>Date:</span> <span>{{ orderData.date }}</span>
                  </div>
                  <div style="display:flex; justify-content:space-between;">
                    <span>Customer:</span> <span>{{ orderData.customerName }}</span>
                  </div>
                </div>

                <div style="border-top:1px dashed #000; margin:6px 0;"></div>

                <!-- ITEMS -->
                <div style="font-size:12px; margin-bottom:8px;">
                  <strong>Items</strong>
                  <div
                    v-for="item in orderData.items"
                    :key="item.id"
                    style="margin-top:4px;"
                  >
                    <div style="display:flex; justify-content:space-between;">
                      <span>{{ item.name }} x{{ item.quantity }}</span>
                      <span>Rp {{ (item.price * item.quantity).toLocaleString() }}</span>
                    </div>
                  </div>
                </div>

                <div style="border-top:1px dashed #000; margin:6px 0;"></div>

                <!-- TOTAL -->
                <div style="font-size:12px; margin-bottom:8px;">
                  <div style="display:flex; justify-content:space-between;">
                    <span>Subtotal:</span>
                    <span>Rp {{ orderData.subtotal.toLocaleString() }}</span>
                  </div>
                  <div style="display:flex; justify-content:space-between;">
                    <span>Tax:</span>
                    <span>Rp {{ orderData.tax.toLocaleString() }}</span>
                  </div>
                  <div
                    style="display:flex; justify-content:space-between; font-weight:bold; font-size:14px; margin-top:4px;"
                  >
                    <span>Total:</span>
                    <span>Rp {{ orderData.total.toLocaleString() }}</span>
                  </div>
                </div>

                <div style="border-top:1px dashed #000; margin:6px 0;"></div>

                <!-- PAYMENT -->
                <div style="font-size:12px; margin-bottom:8px;">
                  <div style="display:flex; justify-content:space-between;">
                    <span>Payment:</span>
                    <span>{{ orderData.paymentMethod }}</span>
                  </div>

                  <div v-if="orderData.paymentMethod === 'cash'" style="margin-top:4px;">
                    <div style="display:flex; justify-content:space-between;">
                      <span>Received:</span>
                      <span>Rp {{ orderData.amountReceived.toLocaleString() }}</span>
                    </div>
                    <div style="display:flex; justify-content:space-between;">
                      <span>Change:</span>
                      <span>Rp {{ orderData.change.toLocaleString() }}</span>
                    </div>
                  </div>
                </div>

                <div style="border-top:1px dashed #000; margin:10px 0;"></div>

                <!-- FOOTER -->
                <div style="text-align:center; font-size:11px;">
                  <p style="margin:2px 0;">Terima kasih telah berkunjung!</p>
                  <p style="margin:2px 0;">{{ outlet.name }}</p>
                </div>
              </div>

              <!-- BUTTONS -->
              <div class="flex space-x-3 mt-6">
                <button
                  @click="printReceipt"
                  class="flex-1 px-4 py-2 text-sm font-medium text-gray-700 bg-gray-100 border border-gray-300 rounded-md hover:bg-gray-200"
                >
                  Print Receipt
                </button>
                <button
                  @click="$emit('close')"
                  class="flex-1 px-4 py-2 text-sm font-medium text-white bg-blue-600 rounded-md hover:bg-blue-700"
                >
                  Close
                </button>
              </div>
            </DialogPanel>
          </TransitionChild>
        </div>
      </div>
    </Dialog>
  </TransitionRoot>
</template>

<script setup>
import {
  Dialog,
  DialogPanel,
  DialogTitle,
  TransitionChild,
  TransitionRoot
} from "@headlessui/vue";
import { CheckCircleIcon } from "@heroicons/vue/24/outline";

defineProps({
  isOpen: Boolean,
  orderData: Object
});

defineEmits(["close"]);

/* ======================================================
   AMBIL OUTLET + KASIR DARI LOCALSTORAGE
====================================================== */

// Default fallback outlet
let outlet = {
  name: "Cafe Fifteenth",
  phone: "081234567891",
  wifi: "Fifteenth15"
};

// Default fallback kasir
let cashierName = "Kasir";

/* Ambil outlet */
if (typeof window !== "undefined") {
  const savedOutlet = localStorage.getItem("selectedOutlet");

  if (savedOutlet) {
    const o = JSON.parse(savedOutlet);
    outlet = {
      name: o.name || outlet.name,
      address: o.address || outlet.address,
      phone: o.phone || outlet.phone,
      wifi: o.wifi || outlet.wifi
    };
  }

  /* Ambil user yang login */
  const userData = localStorage.getItem("userData");

  if (userData) {
    const user = JSON.parse(userData);
    cashierName = user.name || user.username || "Kasir";
  }
}

/* ======================================================
   PRINT RECEIPT
====================================================== */
const printReceipt = () => {
  const receiptContent = document.querySelector(".receipt-content");

  const printWindow = window.open("", "_blank", "width=400,height=600");

  printWindow.document.write(`
    <html>
      <head>
        <title>Receipt</title>

        <style>
          @page {
            size: 58mm auto;
            margin: 0;
          }

          body {
            width: 58mm;
            margin: 0;
            padding: 0 6px;
            font-family: 'Arial', sans-serif;
            font-size: 11px;
          }

          .receipt-content {
            width: 100%;
          }
        </style>

      </head>
      <body>
        ${receiptContent.innerHTML}
      </body>
    </html>
  `);

  printWindow.document.close();
  printWindow.focus();
  printWindow.print();
  printWindow.close();
};
</script>
