<template>
    <div class="login-page">
        <div class="login-form">
            <div class="brand">
                <div class="brand-icon">
                    <svg width="28" height="28" viewBox="0 0 48 48" aria-hidden="true">
                        <path d="M24 4 C30 10, 30 16, 24 18 C18 16, 18 10, 24 4 Z" fill="#07382c"/>
                        <rect x="22" y="2" width="4" height="6" fill="#07382c"/>
                        <path d="M10 40 L10 26 C10 18, 16 14, 24 14 C32 14, 38 18, 38 26 L38 40 Z" fill="#07382c"/>
                        <rect x="6" y="40" width="36" height="4" fill="#07382c"/>
                    </svg>
                </div>
                <h2>Prayers Times</h2>
                <p class="brand-subtitle">Log in to your account</p>
            </div>
 
            <form @submit.prevent="handleLogin">
                <div class="form-group">
                    <label for="username">Username:</label>
                    <input type="text" id="username" v-model="username" />
                </div>
                <div class="form-group">
                    <label for="password">Password:</label>
                    <input type="password" id="password" v-model="password" />
                </div>
                <button type="submit">Login</button>
 
                <p class="register-label">Don't have an account? <RouterLink to="/register">Register</RouterLink></p>
                <p class="register-label"><RouterLink to="/forgot-password">Forgot Password?</RouterLink></p>
            </form>
 
            <div class="dome-decoration">
                <span></span>
                <span></span>
                <span></span>
            </div>
        </div>
    </div>
</template>
 
<script>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useAuthStore } from '../stores/auth';
 
const API_BASE_URL = 'http://localhost:3001/api/auth';
 
export default {
    name: 'LoginForm',
    setup() {
        const username = ref('');
        const password = ref('');
        const router = useRouter();
        const authStore = useAuthStore();
 
        const handleLogin = async () => {
            if (!username.value.trim() || !password.value.trim()) {
                alert('All fields are required');
                return;
            }
 
            try {
                const response = await fetch(`${API_BASE_URL}/login`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        username: username.value,
                        password: password.value,
                    }),
                });
 
                const result = await response.json();
 
                if (!response.ok) {
                    alert(result.message || 'Login gagal.');
                    return;
                }
 
                authStore.login(username.value);
                alert(result.message || 'Login berhasil.');
                router.push('/dashboard');
            } catch (_error) {
                alert('Backend belum berjalan atau terjadi kesalahan jaringan.');
            }
        };
 
        return { username, password, handleLogin };
    }
}
</script>
 
<style scoped>
.login-page {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(160deg, #04241c 0%, #0a4536 60%, #0d5a45 100%);
    padding: 1.5rem;
}
 
.login-form {
    position: relative;
    overflow: hidden;
    max-width: 400px;
    width: 100%;
    margin: 0 auto;
    padding: 2.5rem 2rem 2rem;
    border-radius: 16px;
    background-color: #07382c;
    border: 1px solid #1c5945;
}
 
.login-form::before {
    content: "";
    position: absolute;
    top: -60px;
    left: 50%;
    transform: translateX(-50%);
    width: 220px;
    height: 220px;
    border-radius: 50%;
    background: rgba(212, 175, 55, 0.08);
    pointer-events: none;
}
 
.brand {
    position: relative;
    text-align: center;
    margin-bottom: 1.75rem;
}
 
.brand-icon {
    width: 56px;
    height: 56px;
    margin: 0 auto 0.75rem;
    border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
    background-color: #d4af37;
    display: flex;
    align-items: center;
    justify-content: center;
}
 
.brand h2 {
    margin: 0;
    font-size: 20px;
    font-weight: 600;
    color: #f4e9c1;
    letter-spacing: 0.3px;
}
 
.brand-subtitle {
    margin: 4px 0 0;
    font-size: 13px;
    color: #9fcdb8;
}
 
.register-label {
    display: block;
    margin-top: 1rem;
    text-align: center;
    font-size: 13.5px;
    color: #9fcdb8;
}
 
.register-label a {
    color: #f4e9c1;
    font-weight: 500;
    text-decoration: none;
}
 
.register-label a:hover {
    text-decoration: underline;
}
 
.form-group {
    margin-bottom: 1.1rem;
    position: relative;
}
 
label {
    display: block;
    margin-bottom: 0.4rem;
    font-size: 13.5px;
    color: #cfe8db;
}
 
input {
    width: 100%;
    padding: 0.65rem 0.75rem;
    border: 1px solid #2e6b56;
    border-radius: 8px;
    background-color: #0d4a3a;
    color: #f4e9c1;
    font-size: 14px;
    box-sizing: border-box;
    transition: border-color 0.2s, box-shadow 0.2s;
}
 
input::placeholder {
    color: #5f9783;
}
 
input:focus {
    outline: none;
    border-color: #d4af37;
    box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.15);
}
 
button {
    width: 100%;
    padding: 0.8rem;
    border: none;
    border-radius: 8px;
    background-color: #d4af37;
    color: #07382c;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s;
}
 
button:hover {
    background-color: #c39d2c;
}
 
.dome-decoration {
    position: relative;
    display: flex;
    justify-content: center;
    gap: 6px;
    margin-top: 1.75rem;
}
 
.dome-decoration span {
    width: 28px;
    background-color: rgba(212, 175, 55, 0.22);
    border-top-left-radius: 14px;
    border-top-right-radius: 14px;
}
 
.dome-decoration span:nth-child(1) {
    height: 28px;
}
 
.dome-decoration span:nth-child(2) {
    height: 40px;
    background-color: rgba(212, 175, 55, 0.32);
}
 
.dome-decoration span:nth-child(3) {
    height: 28px;
}
</style>