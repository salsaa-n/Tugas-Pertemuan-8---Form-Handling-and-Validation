<template>
    <div class="register-form">
        <h2>Register</h2>
        <form @submit.prevent="handleRegister">
        <div class="form-group">
            <label for="username">Username:</label>
            <input type="text" id="username" v-model="username" />
        </div>
        <div class="form-group">
            <label for="email">Email:</label>
            <input type="text" id="email" v-model="email" />
        </div>
         <div class="form-group">
            <label for="password">Password:</label>
            <input type="password" id="password" v-model="password"  />
        </div>      
        <div class="form-group">
            <label for="confirm-password">Confirm Password:</label>
            <input type="password" id="confirm-password" v-model="confirmPassword" />
        </div>

        <button type="submit">Register</button>
        <label for="login" class="register-label">Already have an account? <a href="/login">Login</a></label>
        </form>
    </div>
</template>
<script>
import { ref } from 'vue';

export default {
    name: 'RegisterForm',
    setup() {
        const username = ref('');
        const email = ref('');
        const password = ref('');
        const confirmPassword = ref('');

        const handleRegister = () => {
    let message = "";

    if (!username.value.trim()) {
        message = "Username is required!";
    } else if (username.value.length < 3) {
        message = "Username must be at least 3 characters!";
    } else if (/\d/.test(username.value)) {
        message = "Username cannot contain numbers!";
    } else if (!email.value.trim()) {
        message = "Email is required!";
    } else if (!email.value.includes("@")) {
        message = "Invalid email address!";
    } else if (!password.value.trim()) {
        message = "Password is required!";
    } else if (password.value.length < 8) {
        message = "Password must be at least 8 characters!";
    } else if (!/[A-Z]/.test(password.value)) {
        message = "Password must contain at least one uppercase letter!";
    } else if (!/\d/.test(password.value)) {
        message = "Password must contain at least one number!";
    } else if (!confirmPassword.value.trim()) {
        message = "Confirm Password is required!";
    } else if (password.value !== confirmPassword.value) {
        message = "Passwords do not match!";
    }

    if (message) {
        alert(message);
    } else {
        alert("success");
    }
};

        return { username, email, password, confirmPassword, handleRegister };
    }
}
</script>
<style scoped>
.register-label{
    display: block;
    margin-top: 1rem;
    text-align: center;
}
.register-form {
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
</style>