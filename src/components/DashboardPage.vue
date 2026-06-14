<template>
  <section class="dashboard">
    <h2>Dashboard</h2>
    <p>Selamat datang, {{ authStore.username || 'User' }}.</p>

    <button type="button" class="logout-button" @click="handleLogout">
      Logout
    </button>
  </section>
</template>

<script setup>
import { useRouter } from 'vue-router';
import { useAuthStore } from '../stores/auth';

const router = useRouter();
const authStore = useAuthStore();

const handleLogout = () => {
  authStore.logout();
  router.push('/login');
};
</script>

<style scoped>
.dashboard {
  max-width: 500px;
  margin: 0 auto;
  padding: 2rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #f9f9f9;
  text-align: center;
}

h2 {
  margin-bottom: 0.75rem;
}

p {
  margin-bottom: 1.25rem;
}

.logout-button {
  width: 100%;
  padding: 0.75rem;
  border: none;
  border-radius: 4px;
  background-color: #e53e3e;
  color: #fff;
  font-size: 1rem;
  cursor: pointer;
}

.logout-button:hover {
  background-color: #c53030;
}
</style>