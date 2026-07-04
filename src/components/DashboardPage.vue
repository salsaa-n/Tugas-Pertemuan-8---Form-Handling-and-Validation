<template>
  <div class="dashboard-page">

    <!-- Navbar -->
    <nav class="navbar">
      <div class="navbar-brand">
        <svg width="22" height="22" viewBox="0 0 48 48" aria-hidden="true">
          <path d="M24 4 C30 10, 30 16, 24 18 C18 16, 18 10, 24 4 Z" fill="#d4af37"/>
          <rect x="22" y="2" width="4" height="6" fill="#d4af37"/>
          <path d="M10 40 L10 26 C10 18, 16 14, 24 14 C32 14, 38 18, 38 26 L38 40 Z" fill="#d4af37"/>
          <rect x="6" y="40" width="36" height="4" fill="#d4af37"/>
        </svg>
        <span>Prayer Times</span>
      </div>
      <div class="navbar-right">
        <span class="navbar-user">{{ authStore.username || 'User' }}</span>
        <button type="button" class="logout-button" @click="handleLogout">Logout</button>
      </div>
    </nav>

    <!-- Main content -->
    <main class="main-content">
      <p class="welcome-text">Welcome, {{ authStore.username || 'User' }} 👋</p>

      <!-- Komponen PrayerTimes -->
      <PrayerTimes />

      
    </main>

  </div>
</template>

<script setup>
import { useRouter } from 'vue-router';
import { useAuthStore } from '../stores/auth';
import PrayerTimes from './PrayerTimes.vue';


const router = useRouter();
const authStore = useAuthStore();

const handleLogout = () => {
  authStore.logout();
  router.push('/login');
};
</script>

<style scoped>
.dashboard-page {
  min-height: 100vh;
  background: linear-gradient(160deg, #04241c 0%, #0a4536 60%, #0d5a45 100%);
  display: flex;
  flex-direction: column;
}

.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.9rem 2rem;
  background-color: rgba(4, 36, 28, 0.85);
  backdrop-filter: blur(8px);
  border-bottom: 1px solid #1c5945;
  position: sticky;
  top: 0;
  z-index: 10;
}

.navbar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.1rem;
  font-weight: 600;
  color: #f4e9c1;
}

.navbar-right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.navbar-user {
  font-size: 0.9rem;
  color: #9fcdb8;
}

.logout-button {
  padding: 0.45rem 1rem;
  border: 1px solid #e53e3e;
  border-radius: 6px;
  background-color: transparent;
  color: #e53e3e;
  font-size: 0.85rem;
  cursor: pointer;
  transition: background-color 0.2s, color 0.2s;
}

.logout-button:hover {
  background-color: #e53e3e;
  color: #fff;
}

.main-content {
  max-width: 860px;
  width: 100%;
  margin: 2rem auto;
  padding: 0 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.welcome-text {
  margin: 0;
  font-size: 1.1rem;
  color: #f4e9c1;
}
</style>