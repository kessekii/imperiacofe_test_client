<template>
    <div class="login-page">
      <h1>Login</h1>
      <form @submit.prevent="handleLogin" class="login-form">
        <div class="form-group">
          <label for="username">Username:</label>
          <input id="username" v-model="username" type="text" required />
        </div>
        <div class="form-group">
          <label for="password">Password:</label>
          <input id="password" v-model="password" type="password" required />
        </div>
        <button type="submit" class="btn btn-login">Login</button>
      </form>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    </div>
  </template>
  
  <script lang="ts" setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
  
  const username = ref<string>('');
  const password = ref<string>('');
  const errorMessage = ref<string | null>(null);
  const router = useRouter();
  // API Base URL
  window.localStorage.setItem('token', '');
const API_BASE_URL = 'http://localhost:3400/auth';

  async function handleLogin(): Promise<void> {
    try {
      const response = await axios.post(API_BASE_URL + '/login', {
        username: username.value,
        password: password.value,
      });
      if (response.data.access_token) {
        // Store the token in local storage
        window.localStorage.setItem('token', response.data.access_token);
        // Redirect to the product management page
        router.push('/products');
      } else {
        errorMessage.value = 'Invalid username or password';
      }
    } catch (error) {
      console.error('Login error:', error);
      errorMessage.value = 'An error occurred during login';
    }
  }
  </script>
  
  <style lang="scss" scoped>
  .login-page {
    max-width: 400px;
    margin: 50px auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #f9f9f9;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    text-align: center;
  
    h1 {
      margin-bottom: 20px;
      color: #333;
    }
  
    .login-form {
      display: flex;
      flex-direction: column;
  
      .form-group {
        margin-bottom: 15px;
  
        label {
          font-weight: bold;
          margin-bottom: 5px;
          display: block;
        }
  
        input {
          width: 100%;
          padding: 10px;
          border: 1px solid #ddd;
          border-radius: 4px;
          font-size: 16px;
        }
      }
  
      .btn-login {
        background-color: #007bff;
        color: white;
        padding: 10px 15px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 16px;
  
        &:hover {
          background-color: #0056b3;
        }
      }
    }
  
    .error-message {
      color: #dc3545;
      margin-top: 10px;
    }
  }
  </style>