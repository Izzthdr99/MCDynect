<script setup>
/**
 * DashboardSidebar.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1214-2961
 *
 * Primary navigation for the SuperAdmin dashboard.
 */
import iconFullLogo from '../assets/dashboard/sidebar/full-logo.svg'
import iconDashboard from '../assets/dashboard/sidebar/icon-dashboard.svg'
import iconStaffRoles from '../assets/dashboard/sidebar/icon-staff-roles.svg'
import iconSettings from '../assets/dashboard/sidebar/icon-settings.svg'

defineProps({
  activeItem: {
    type: String,
    default: 'dashboard',
  },
  collapsed: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['navigate'])

// Each icon keeps its own exported Figma size (not a forced 20x20 square) —
// the source SVGs are cropped to the vector's own bounding box, so scaling
// them all to the same square distorts the ones that aren't already square.
const navItems = [
  { key: 'dashboard', icon: iconDashboard, label: 'Dashboard', width: 15, height: 15 },
  { key: 'staff-roles', icon: iconStaffRoles, label: 'Staff & Roles', width: 16.67, height: 13.33 },
  { key: 'system-settings', icon: iconSettings, label: 'System Settings', width: 16.67, height: 16.67 },
]

const settingsIcon = { icon: iconSettings, width: 16.67, height: 16.67 }
</script>

<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-header">
      <img class="logo" :src="iconFullLogo" alt="MC Dynect" />
    </div>

    <nav class="sidebar-nav">
      <button
        v-for="item in navItems"
        :key="item.key"
        type="button"
        class="nav-item"
        :class="{ active: activeItem === item.key }"
        :title="collapsed ? item.label : undefined"
        @click="emit('navigate', item.key)"
      >
        <span class="nav-icon-slot">
          <img :src="item.icon" :style="{ width: item.width + 'px', height: item.height + 'px' }" alt="" />
        </span>
        <span class="nav-label">{{ item.label }}</span>
        <span class="nav-tooltip" role="tooltip">{{ item.label }}</span>
      </button>
    </nav>

    <div class="sidebar-footer">
      <button
        type="button"
        class="nav-item"
        :title="collapsed ? 'Settings' : undefined"
        @click="emit('navigate', 'settings')"
      >
        <span class="nav-icon-slot">
          <img
            :src="settingsIcon.icon"
            :style="{ width: settingsIcon.width + 'px', height: settingsIcon.height + 'px' }"
            alt=""
          />
        </span>
        <span class="nav-label">Settings</span>
        <span class="nav-tooltip" role="tooltip">Settings</span>
      </button>
    </div>
  </aside>
</template>

<style scoped>
.sidebar {
  --nav-active-bg: #F9E2E5;
  --nav-active-text: #BD2841;
  --nav-text: #4a5565;
  --border: #e5e7eb;

  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  width: 251px;
  height: 100vh;
  background: #ffffff;
  border-right: 1px solid var(--border);
  overflow-y: auto;
  transition: width 220ms ease;
  z-index: 20;
}
.sidebar.collapsed {
  width: 72px;
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 27px 7px 19px;
  border-bottom: 1px solid var(--border);
}

.logo {
  height: 20px;
  width: auto;
  max-width: 130px;
  opacity: 1;
  transition: opacity 160ms ease, max-width 220ms ease;
}
.sidebar.collapsed .logo {
  opacity: 0;
  max-width: 0;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: 8px;
  flex: 1;
  padding: 8px;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 12px 8px;
  border-top: 1px solid var(--border);
}

.nav-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 10px 12px;
  border: none;
  border-radius: 8px;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: var(--nav-text);
  text-align: left;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease;
}
.nav-item:hover {
  background: #F9E2E5;
}
.nav-item.active {
  background: var(--nav-active-bg);
  color: var(--nav-active-text);
}

.nav-icon-slot {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}
</style>
