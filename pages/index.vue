<script setup>
if (process.client) {
  const isLoggedIn = localStorage.getItem("isLoggedIn");

  if (isLoggedIn !== "true") {
    navigateTo("/login");
  }
}
</script>

<template>
  <div></div>
</template>
