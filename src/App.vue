<script setup>
import { ref } from 'vue'
import McDynectLogin from './pages/McDynectLogin.vue'
import McDynectWorkspace from './pages/McDynectWorkspace.vue'
import SuperAdminDashboard from './pages/SuperAdminDashboard.vue'
import LicensingDashboard from './pages/LicensingDashboard.vue'
import FinanceDashboard from './pages/FinanceDashboard.vue'

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
  if (moduleKey === 'licensing') {
    currentView.value = 'licensing-dashboard'
    return
  }
  if (moduleKey === 'finance') {
    currentView.value = 'finance-dashboard'
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
    @sign-out="currentView = 'login'"
  />
  <LicensingDashboard
    v-else-if="currentView === 'licensing-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
    @sign-out="currentView = 'login'"
  />
  <FinanceDashboard
    v-else-if="currentView === 'finance-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
    @sign-out="currentView = 'login'"
  />
  <McDynectWorkspace v-else @select="handleModuleSelect" />
</template>
