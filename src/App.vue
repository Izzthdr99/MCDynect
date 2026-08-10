<script setup>
import McDynectLogin from './components/McDynectLogin.vue'

function handleLogin({ email, password, rememberMe }) {
  console.log('login submitted', { email, password, rememberMe })
}
</script>

<template>
  <McDynectLogin @submit="handleLogin" />
</template>
