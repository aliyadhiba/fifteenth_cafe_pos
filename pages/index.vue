<script setup>
import { onMounted } from 'vue'
import { navigateTo } from '#app'

onMounted(() => {
  // pastikan hanya jalan di client
  if (process.server) return

  const isLoggedIn = localStorage.getItem('isLoggedIn')

  if (isLoggedIn === 'true') {
    navigateTo('/outlet') // atau '/pos' sesuai alur kamu
  } else {
    navigateTo('/login')
  }
})
</script>

<template>
  <!-- root tidak menampilkan UI -->
  <div></div>
</template>
