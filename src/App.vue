<script setup>
import { computed, ref } from 'vue'
import McDynectLogin from './pages/McDynectLogin.vue'
import McDynectWorkspace from './pages/McDynectWorkspace.vue'
import SuperAdminDashboard from './pages/SuperAdminDashboard.vue'
import LicensingDashboard from './pages/LicensingDashboard.vue'
import FinanceDashboard from './pages/FinanceDashboard.vue'
import ProcurementDashboard from './pages/ProcurementDashboard.vue'
import OutletManagerDashboard from './pages/OutletManagerDashboard.vue'
import InventoryDashboard from './pages/InventoryDashboard.vue'
import LicenseeDashboard from './pages/LicenseeDashboard.vue'
import AiChatPanel from './components/AiChatPanel.vue'

const currentView = ref('login')

// AI chat panel is global, except on the login and workspace-selection screens.
const showChatPanel = computed(() => currentView.value !== 'login' && currentView.value !== 'workspace')

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
  if (moduleKey === 'procurement') {
    currentView.value = 'procurement-dashboard'
    return
  }
  if (moduleKey === 'operation-manager') {
    currentView.value = 'outlet-manager-dashboard'
    return
  }
  if (moduleKey === 'inventory') {
    currentView.value = 'inventory-dashboard'
    return
  }
  if (moduleKey === 'licensee') {
    currentView.value = 'licensee-dashboard'
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
  <ProcurementDashboard
    v-else-if="currentView === 'procurement-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
    @sign-out="currentView = 'login'"
  />
  <OutletManagerDashboard
    v-else-if="currentView === 'outlet-manager-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
    @sign-out="currentView = 'login'"
  />
  <InventoryDashboard
    v-else-if="currentView === 'inventory-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
    @sign-out="currentView = 'login'"
  />
  <LicenseeDashboard
    v-else-if="currentView === 'licensee-dashboard'"
    @back-to-workspace="currentView = 'workspace'"
    @sign-out="currentView = 'login'"
  />
  <McDynectWorkspace v-else @select="handleModuleSelect" />

  <AiChatPanel v-if="showChatPanel" />
</template>
