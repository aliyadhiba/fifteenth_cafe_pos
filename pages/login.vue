<template>
  <div class="login-page">
    <div class="login-box">
      <!-- Logo -->
      <div class="logo">
        <div class="icon-placeholder"></div>
      </div>

      <h2 class="title">Sign in to continue</h2>

      <!-- Form Login -->
      <form @submit.prevent="handleLogin" class="login-form">
        <div class="input-group">
          <input type="email" v-model="email" placeholder="Email" required />
        </div>
        <div class="input-group">
          <input type="password" v-model="password" placeholder="Password" required />
        </div>

        <button type="submit" class="btn-login">Sign In</button>
      </form>

      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { navigateTo } from "#app";

definePageMeta({
  layout: "login",
});

const email = ref("");
const password = ref("");
const errorMessage = ref("");

/* ======================
   DATA PEGAWAI LOGIN
   ====================== */
const users = [
  {
    email: "admin@gmail.com",
    password: "123",
    name: "Admin Fifteenth",
    dob: "2000-01-01",
    gender: "Male",
    username: "admin",
    photo: null,
  },
  {
    email: "kasir1@gmail.com",
    password: "111",
    name: "Kasir 1",
    dob: "2001-02-11",
    gender: "Female",
    username: "kasir1",
    photo: null,
  },
  {
    email: "kasir2@gmail.com",
    password: "222",
    name: "Kasir 2",
    dob: "2002-07-20",
    gender: "Male",
    username: "kasir2",
    photo: null,
  },
];

/* ======================
   LOGIN FUNCTION
   ====================== */
const handleLogin = () => {
  const user = users.find(
    (u) => u.email === email.value && u.password === password.value
  );

  if (user) {
    // Status login
    localStorage.setItem("isLoggedIn", "true");

    // Simpan informasi user
    localStorage.setItem("currentUser", user.name);
    localStorage.setItem("employeeProfile", JSON.stringify(user));

    // WAJIB untuk struk dan profile
    localStorage.setItem(
      "userData",
      JSON.stringify({
        name: user.name,
        username: user.username,
        email: user.email,
        gender: user.gender,
        dob: user.dob,
        photo: user.photo,
      })
    );

    // Reset outlet yang lama
    localStorage.removeItem("currentOutlet");
    localStorage.removeItem("selectedOutlet");

    // Redirect ke pilihan outlet
    navigateTo("/outlet");
  } else {
    errorMessage.value = "Email atau password salah!";
  }
};
</script>

<style scoped>
.login-page {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: #f7f7f7;
}

.login-box {
  background: white;
  padding: 2.5rem 3rem;
  border-radius: 12px;
  width: 360px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
  text-align: center;
}

.logo {
  margin-bottom: 1.5rem;
}

.icon-placeholder {
  width: 50px;
  height: 50px;
  background: #ccc;
  border-radius: 12px;
  margin: 0 auto;
}

.title {
  font-size: 1.5rem;
  font-weight: bold;
  color: #333;
  margin-bottom: 0.5rem;
}

.input-group {
  margin-bottom: 1rem;
}

input {
  width: 100%;
  padding: 0.75rem;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 1rem;
}

.btn-login {
  width: 100%;
  padding: 0.75rem;
  border-radius: 8px;
  border: none;
  background: #000;
  color: white;
  font-weight: bold;
  cursor: pointer;
  margin-top: 1rem;
  transition: background 0.3s;
}

.btn-login:hover {
  background: #333;
}

.error {
  color: red;
  margin-top: 1rem;
  font-weight: bold;
}
</style>
