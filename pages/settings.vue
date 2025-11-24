<template>
  <div class="settings-wrapper">
    <main class="content">
      <!-- Profile Picture Section -->
      <section class="section">
        <h3 class="section-title">Profile Picture</h3>

        <div class="photo-row">
          <img :src="previewImage || defaultPhoto" class="profile-photo" alt="profile" />

          <div class="photo-actions">
            <label class="upload-btn">
              Upload new picture
              <input type="file" accept="image/*" @change="onFileChange" hidden />
            </label>

            <button class="remove-btn" @click="removePhoto">Remove</button>
          </div>
        </div>

        <!-- Profile details -->
        <div class="item-list">
          <div class="item" @click="edit('name')">
            <span>Name</span>
            <span class="value">{{ profile.name }}</span>
            <span class="arrow">›</span>
          </div>

          <div class="item" @click="edit('dob')">
            <span>Date of Birth</span>
            <span class="value">{{ profile.dob }}</span>
            <span class="arrow">›</span>
          </div>

          <div class="item" @click="edit('gender')">
            <span>Gender</span>
            <span class="value">{{ profile.gender }}</span>
            <span class="arrow">›</span>
          </div>

          <div class="item" @click="edit('email')">
            <span>Email</span>
            <span class="value">{{ profile.email }}</span>
            <span class="arrow">›</span>
          </div>
        </div>
      </section>

      <!-- Account Info Section -->
      <section class="section">
        <h3 class="section-title">Account info</h3>

        <div class="item-list">
          <div class="item" @click="edit('username')">
            <span>Username</span>
            <span class="value">{{ profile.username }}</span>
            <span class="arrow">›</span>
          </div>

          <div class="item" @click="edit('password')">
            <span>Password</span>
            <span class="value">********</span>
            <span class="arrow">›</span>
          </div>
        </div>
      </section>
    </main>

    <!-- FOOTER BAR -->
    <div class="footer-bar">Fifteenth</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const defaultPhoto = 'https://cdn-icons-png.flaticon.com/512/149/149071.png';
const previewImage = ref(null);

// Properly loaded user profile (FIX)
const profile = ref({
  name: '',
  dob: '',
  gender: '',
  email: '',
  username: '',
});

// Load user from correct localStorage key
onMounted(() => {
  const savedUser = localStorage.getItem('employeeProfile');

  if (savedUser) {
    const user = JSON.parse(savedUser);

    profile.value = {
      name: user.name || '',
      dob: user.dob || '',
      gender: user.gender || '',
      email: user.email || '',
      username: user.username || '',
    };

    if (user.photo) {
      previewImage.value = user.photo;
    }
  }
});

// Upload preview
const onFileChange = (event) => {
  const file = event.target.files?.[0];
  if (file) previewImage.value = URL.createObjectURL(file);
};

// Remove preview
const removePhoto = () => {
  previewImage.value = null;
};

const edit = (field) => {
  alert(`Edit field: ${field}`);
};
</script>

<style scoped>
/* Wrapper buat memastikan footer di bawah dan konten scrollable */
.settings-wrapper {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(to bottom right, #fff, #ffe8f1);
}

/* Konten utama (scrollable jika panjang) */
.content {
  padding: 40px;
  padding-bottom: 20px;
  flex: 1 1 auto;
  overflow-y: auto;
  margin-left: 280px;
}

/* Section card */
.section {
  background: white;
  padding: 25px;
  border-radius: 12px;
  margin-bottom: 20px;
  box-shadow: 0 6px 18px rgba(0,0,0,0.04);
}

.section-title {
  font-size: 16px;
  font-weight: 700;
  margin-bottom: 16px;
}

/* Photo row */
.photo-row {
  display: flex;
  gap: 18px;
  align-items: center;
}

.profile-photo {
  width: 72px;
  height: 72px;
  object-fit: cover;
  border-radius: 999px;
  border: 2px solid #eee;
}

.photo-actions { display: flex; flex-direction: column; gap: 6px; }

.upload-btn {
  color: #175cd3;
  cursor: pointer;
  font-weight: 600;
}

.remove-btn {
  background: none;
  border: none;
  color: #e02424;
  cursor: pointer;
}

.item-list { margin-top: 20px; }
.item {
  display: grid;
  grid-template-columns: 1fr auto auto;
  align-items: center;
  padding: 14px 0;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
}
.item:hover { background: #fafafa; }
.item .value { color: #333; margin-right: 8px; }
.arrow { color: rgba(0,0,0,0.35); font-size: 18px; }

@media (max-width: 900px) {
  .content { margin-left: 0; padding: 20px; }
  .profile-photo { width: 64px; height: 64px; }
}

.footer-bar {
  height: 60px;
  background: #175cd3;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 18px;
  position: fixed;
  bottom: 0;
  width: 100%;
}
</style>
