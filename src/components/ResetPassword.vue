<template>
    <div class="reset-password-form">
        <h2>Reset Password</h2>
        <form @submit.prevent="handleResetPassword">
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="text" id="email" v-model="email" />
            </div>
            <div class="form-group">
                <label for="newPassword">New Password:</label>
                <input type="password" id="newPassword" v-model="newPassword" />
            </div>
            <div class="form-group">
                <label for="confirmPassword">Confirm Password:</label>
                <input type="password" id="confirmPassword" v-model="confirmPassword" />
            </div>
            <button type="submit">Reset Password</button>
            <label class="link-label">Remember your password? <a href="/login">Login</a></label>
        </form>
    </div>
</template>
<script>
import { ref } from 'vue';

export default {
    name: 'ResetPassword',

    setup() {
        const email = ref('');
        const newPassword = ref('');
        const confirmPassword = ref('');

        const handleResetPassword = () => {
            let message = "";

            const emailRule = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

            if (!email.value.trim()) {
                message = "Email is required!";
            } else if (!emailRule.test(email.value)) {
                message = "Please enter a valid email!";
            } else if (!newPassword.value.trim()) {
                message = "New Password is required!";
            } else if (newPassword.value.length < 8) {
                message = "Password must be at least 8 characters!";
            } else if (!/[A-Z]/.test(newPassword.value)) {
                message = "Password must contain at least one uppercase letter!";
            } else if (!/\d/.test(newPassword.value)) {
                message = "Password must contain at least one number!";
            } else if (!confirmPassword.value.trim()) {
                message = "Confirm Password is required!";
            } else if (newPassword.value !== confirmPassword.value) {
                message = "Passwords do not match!";
            }

            if (message) {
                alert(message);
            } else {
                alert("success");
            }
        };

        return {
            email,
            newPassword,
            confirmPassword,
            handleResetPassword
        };
    }
}
</script>
<style scoped>
.reset-password-form {
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