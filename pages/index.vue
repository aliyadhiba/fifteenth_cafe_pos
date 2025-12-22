<script setup>
import { onMounted } from 'vue'
import { navigateTo } from '#app'

onMounted(() => {
  if (process.server) return

  const isLoggedIn = localStorage.getItem('isLoggedIn')
  const selectedOutlet = localStorage.getItem('selectedOutlet')

  if (isLoggedIn !== 'true') {
    navigateTo('/login')
    return
  }

  if (!selectedOutlet) {
    navigateTo('/outlet')
    return
  }

  navigateTo('/pos')
})
</script>

<template>
  <div></div>
</template>
