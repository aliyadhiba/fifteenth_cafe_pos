<template>
  <div class="flex h-screen bg-gray-50 relative">

    <!-- Sidebar -->
    <div 
      class="fixed inset-y-0 left-0 z-20 w-64 bg-white shadow-lg transform transition-transform duration-300"
      :class="{ '-translate-x-full': !isSidebarOpen }"
    >
      <div class="p-6 border-b border-gray-200 flex items-center justify-between">
        <h1 class="text-xl font-bold text-gray-800">POS System</h1>
      </div>

      <nav class="mt-6">
        <div class="px-4 space-y-2">
          <NuxtLink 
  to="/" 
  @click="closeSidebar"
  class="flex items-center px-4 py-3 text-gray-700 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition"
  :class="{ 'bg-blue-50 text-blue-600 border-r-2 border-blue-600': $route.path === '/' }"
>
  <ShoppingCartIcon class="w-5 h-5 mr-3" />
  POS
</NuxtLink>

<NuxtLink 
  to="/history" 
  @click="closeSidebar"
  class="flex items-center px-4 py-3 text-gray-700 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition"
  :class="{ 'bg-blue-50 text-blue-600 border-r-2 border-blue-600': $route.path === '/history' }"
>
  <ClockIcon class="w-5 h-5 mr-3" />
  History Orders
</NuxtLink>

<NuxtLink 
  to="/activity" 
  @click="closeSidebar"
  class="flex items-center px-4 py-3 text-gray-700 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition"
  :class="{ 'bg-blue-50 text-blue-600 border-r-2 border-blue-600': $route.path === '/activity' }"
>
  <ChartBarIcon class="w-5 h-5 mr-3" />
  Activity
</NuxtLink>

<NuxtLink 
  to="/settings" 
  @click="closeSidebar"
  class="flex items-center px-4 py-3 text-gray-700 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition"
  :class="{ 'bg-blue-50 text-blue-600 border-r-2 border-blue-600': $route.path === '/settings' }"
>
  <CogIcon class="w-5 h-5 mr-3" />
  Settings
</NuxtLink>
        </div>

        <div class="mt-8 px-4">
          <button 
            @click="handleLogout"
            class="flex items-center w-full px-4 py-3 text-red-600 rounded-lg hover:bg-red-50 transition"
          >
            <ArrowRightOnRectangleIcon class="w-5 h-5 mr-3" />
            Logout
          </button>
        </div>
      </nav>
    </div>

    <!-- Main Content -->
    <div 
      class="flex-1 pb-12 overflow-hidden transition-all duration-300"
      :class="{ 'ml-64': isSidebarOpen }"
    >
      <slot />
    </div>

    <!-- FOOTER (TIDAK ADA BACKGROUND BIRU LAGI) -->
    <div 
      class="fixed bottom-0 left-0 w-full h-12 flex items-center justify-between px-4 z-30 bg-transparent"
      :class="{ 'pl-64': isSidebarOpen }"
    >
      <!-- Tombol toggle (HANYA TOMBOL YANG BIRU) -->
      <button 
  @click="isSidebarOpen = !isSidebarOpen"
  class="p-2 bg-blue-600 text-white rounded-md shadow hover:bg-blue-700 transition"
>
  <svg 
    xmlns="http://www.w3.org/2000/svg" 
    class="w-6 h-6 transition-transform"
    fill="none" 
    viewBox="0 0 24 24" 
    stroke="currentColor"
    :class="{ 'rotate-90': isSidebarOpen }"
  >
    <path 
      stroke-linecap="round" 
      stroke-linejoin="round" 
      stroke-width="3.5" 
      d="M4 6h16M4 12h16M4 18h16" 
    />
  </svg>
</button>



      <!-- Spacer -->
      <div class="w-10"></div>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue'
import { 
  ShoppingCartIcon, 
  ClockIcon, 
  ChartBarIcon, 
  CogIcon, 
  ArrowRightOnRectangleIcon 
} from '@heroicons/vue/24/outline'
import { useRouter } from 'vue-router'

const router = useRouter()
const isSidebarOpen = ref(true)

const handleLogout = () => {
  localStorage.removeItem('isLoggedIn')
  router.push('/login')
}
const closeSidebar = () => {
  isSidebarOpen.value = false
}
</script>
