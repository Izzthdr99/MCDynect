<script setup>
import { ref } from 'vue'
import McDynectLogin from './pages/McDynectLogin.vue'
import McDynectWorkspace from './pages/McDynectWorkspace.vue'
import SuperAdminDashboard from './pages/SuperAdminDashboard.vue'

const currentView = ref('login')

function handleLogin({ email, password, rememberMe }) {
  console.log('login submitted', { email, password, rememberMe })
  currentView.value = 'workspace'
}

function handleModuleSelect(moduleKey) {
  if (moduleKey === 'super-admin') {
    currentView.value = 'super-admin-dashboard'
    return
  }
  console.log('module selected', moduleKey)
}
</script>

<template>
  <McDynectLogin v-if="currentView === 'login'" @submit="handleLogin" />
  <SuperAdminDashboard
    v-else-if="currentView === 'super-admin-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
  />
  <McDynectWorkspace v-else @select="handleModuleSelect" />
</template>
