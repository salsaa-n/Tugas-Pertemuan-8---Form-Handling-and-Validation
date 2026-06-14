<template>
    <div class="forgot-password-form">
        <h2>Forgot Password</h2>
        <form @submit.prevent="handleForgotPassword">
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="text" id="email" v-model="email" />
            </div>
            <button type="submit">Send Reset Link</button>

            <label class="link-label">Remember your password? <a href="/login">Login</a></label>
            <label class="link-label">Don't have an account? <a href="/register">Register</a></label>
            
        </form>
    </div>
</template>
<script>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

export default {
    name: 'ForgotPassword',

    setup() {
        const email = ref('');
        const router = useRouter();

        const handleForgotPassword = () => {
            let message = "";
            const emailRule = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

            if (!email.value.trim()) {
                message = "Email is required!";
            } else if (!emailRule.test(email.value)) {
                message = "Please enter a valid email!";
            }

            if (message) {
                alert(message);
                return;
            }

            alert("success");

            router.push({
                name: 'reset-password'
            });
        };

        return {
            email,
            handleForgotPassword
        };
    }
}
</script>
<style scoped>
.forgot-password-form {
    max-width: 400px;
    margin: 0 auto;
    padding: 2rem;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
}
.form-group {
    margin-bottom: 1rem;
}
label {
    display: block;
    margin-bottom: 0.5rem;
}
input {
    width: 100%;
    padding: 0.5rem;        
    border: 1px solid #ccc;
    border-radius: 4px;
}
button {
    width: 100%;
    padding: 0.75rem;
    border: none;
    border-radius: 4px;
    background-color: #42b983; 
    color: white;
    font-size: 1rem;
    cursor: pointer;
}
button:hover {
    background-color: #38a173;
}
.link-label {
    display: block;
    margin-top: 1rem;
    text-align: center;
}
</style>