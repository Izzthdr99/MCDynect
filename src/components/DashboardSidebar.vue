<script setup>
/**
 * DashboardSidebar.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1214-2961
 * Licensing module variant: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1370-4063
 *
 * Primary navigation for dashboard modules. The nav item list (icons,
 * labels, optional right-chevron for not-yet-built sub-navigation) is
 * supplied by the caller via the navItems prop so each module can show
 * its own menu while sharing this shell.
 */
import iconFullLogo from '../assets/dashboard/sidebar/full-logo.svg'
import iconSettings from '../assets/dashboard/sidebar/icon-settings.svg'
import iconChevronRight from '../assets/dashboard/sidebar/licensing/icon-angle-right.svg'

const props = defineProps({
  activeItem: {
    type: String,
    default: 'dashboard',
  },
  collapsed: {
    type: Boolean,
    default: false,
  },
  navItems: {
    type: Array,
    required: true,
  },
})

const emit = defineEmits(['navigate'])

const settingsIcon = { icon: iconSettings, width: 16.67, height: 16.67 }
</script>

<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-header">
      <img class="logo" :src="iconFullLogo" alt="MC Dynect" />
      <span class="logo-mark" aria-hidden="true"><span class="logo-mark-m">M</span><span class="logo-mark-c">C</span></span>
    </div>

    <nav class="sidebar-nav">
      <button
        v-for="item in props.navItems"
        :key="item.key"
        type="button"
        class="nav-item"
        :class="{ active: activeItem === item.key }"
        :title="collapsed ? item.label : undefined"
        @click="emit('navigate', item.key)"
      >
        <span class="nav-icon-slot">
          <img class="nav-icon" :src="item.icon" :style="{ width: item.width + 'px', height: item.height + 'px' }" alt="" />
        </span>
        <span class="nav-label">{{ item.label }}</span>
        <img v-if="item.chevron" class="nav-chevron" :src="iconChevronRight" alt="" />
        <span class="nav-tooltip" role="tooltip" aria-hidden="true">{{ item.label }}</span>
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
            class="nav-icon"
            :src="settingsIcon.icon"
            :style="{ width: settingsIcon.width + 'px', height: settingsIcon.height + 'px' }"
            alt=""
          />
        </span>
        <span class="nav-label">Settings</span>
        <span class="nav-tooltip" role="tooltip" aria-hidden="true">Settings</span>
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
  transition: width 220ms ease;
  z-index: 20;
}
.sidebar.collapsed {
  width: 64px;
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 66px;
  padding: 0 7px;
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

.logo-mark {
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: inherit;
  font-size: 17px;
  font-weight: 700;
  letter-spacing: -0.5px;
  opacity: 0;
  max-width: 0;
  overflow: hidden;
  transition: opacity 160ms ease, max-width 220ms ease;
}
.sidebar.collapsed .logo-mark {
  opacity: 1;
  max-width: 40px;
}
.logo-mark-m {
  color: #EEBA5D;
}
.logo-mark-c {
  color: #D53F52;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: 0px;
  flex: 1;
  padding: 8px;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: 0px;
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
  transition: background 0.15s ease, color 0.15s ease, gap 220ms ease;
}
.nav-item:hover {
  background: rgba(249, 223, 227, 0.5);
  color: var(--nav-active-text);
}
.nav-item.active {
  background: var(--nav-active-bg);
  color: var(--nav-active-text);
}
.nav-item:hover .nav-icon,
.nav-item.active .nav-icon {
  /* Recolors the #4A5565 source icon to the primary #BD2841, calibrated to an exact match. */
  filter: invert(16%) sepia(79%) saturate(217%) hue-rotate(300deg) brightness(93%) contrast(269%);
}

.nav-icon-slot {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.nav-label {
  overflow: hidden;
  white-space: nowrap;
  max-width: 160px;
  opacity: 1;
  transition: opacity 160ms ease, max-width 220ms ease;
}
.sidebar.collapsed .nav-label {
  opacity: 0;
  max-width: 0;
}

.nav-chevron {
  flex-shrink: 0;
  width: 10.67px;
  height: 6px;
  margin-left: auto;
  transform: rotate(90deg);
  opacity: 1;
  transition: opacity 160ms ease;
}
.sidebar.collapsed .nav-chevron {
  opacity: 0;
  width: 0;
}

.sidebar.collapsed .sidebar-nav {
  gap: 0px;
}

.sidebar.collapsed .nav-item {
  justify-content: center;
  padding: 12px;
  gap: 0;
}

.nav-tooltip {
  position: absolute;
  left: calc(100% + 12px);
  top: 50%;
  padding: 6px 10px;
  border-radius: 6px;
  background: #101828;
  color: #ffffff;
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
  transform: translate(-6px, -50%);
  transition: opacity 160ms ease, transform 160ms ease, visibility 160ms;
  z-index: 30;
}
.sidebar.collapsed .nav-item:hover .nav-tooltip,
.sidebar.collapsed .nav-item:focus-visible .nav-tooltip {
  opacity: 1;
  visibility: visible;
  transform: translate(0, -50%);
}
</style>
