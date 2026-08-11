<script setup>
/**
 * DashboardTopnav.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1232-3424
 * Workspace dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-8647
 * Notification dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-7280
 * Account dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-8006
 *
 * Top bar for the SuperAdmin dashboard: sidebar toggle, workspace switcher,
 * notifications, and the signed-in user's avatar.
 */
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import iconSidebarToggle from '../assets/dashboard/topnav/icon-sidebar-toggle.svg'
import iconChevronSort from '../assets/dashboard/topnav/icon-chevron-sort.svg'
import iconBell from '../assets/dashboard/topnav/icon-bell.svg'
import iconClock from '../assets/dashboard/topnav/icon-clock.svg'
import iconTrash from '../assets/dashboard/topnav/icon-trash.svg'
import iconArrowRight from '../assets/dashboard/topnav/icon-arrow-right.svg'
import iconUser from '../assets/dashboard/topnav/icon-user.svg'
import iconQuestionCircle from '../assets/dashboard/topnav/icon-question-circle.svg'
import iconSignOut from '../assets/dashboard/topnav/icon-sign-out.svg'
import imgAvatar from '../assets/dashboard/topnav/avatar.png'

defineProps({
  collapsed: {
    type: Boolean,
    default: false,
  },
  activeModule: {
    type: String,
    default: null,
  },
})

const emit = defineEmits(['toggle-sidebar', 'switch-workspace', 'open-notifications', 'sign-out'])

const modules = [
  { key: 'finance', title: 'Finance' },
  { key: 'procurement', title: 'Procurement' },
  { key: 'production', title: 'Production' },
  { key: 'inventory', title: 'Inventory' },
  { key: 'licensee', title: 'Licensee' },
  { key: 'licensing', title: 'Licensing' },
  { key: 'compliance', title: 'Compliance' },
  { key: 'outlet-manager', title: 'Outlet Manager' },
  { key: 'area-manager', title: 'Area Manager' },
  { key: 'temporary', title: 'Temporary' },
]

const workspaceMenuOpen = ref(false)
const workspaceSwitcherRef = ref(null)

function toggleWorkspaceMenu() {
  workspaceMenuOpen.value = !workspaceMenuOpen.value
}

function selectModule(moduleKey) {
  workspaceMenuOpen.value = false
  emit('switch-workspace', moduleKey)
}

const notificationsMenuOpen = ref(false)
const notificationsMenuRef = ref(null)
const notificationsTab = ref('all')

const notifications = ref([
  {
    id: 1,
    category: 'Procurement',
    message: 'Your performance review is due next week.',
    time: 'just now',
    read: false,
  },
  {
    id: 2,
    category: 'Finance',
    message: "Don't forget to complete your training modules by Friday!",
    time: '5 minutes ago',
    read: false,
  },
  {
    id: 3,
    category: 'Admin',
    message: 'New healthcare benefits information available, please check your email.',
    time: '10 minutes ago',
    read: true,
  },
  {
    id: 4,
    category: 'Procurement',
    message: 'Your request for redeemable item has been approved!',
    time: '15 minutes ago',
    read: true,
  },
])

const unreadCount = computed(() => notifications.value.filter((item) => !item.read).length)
const visibleNotifications = computed(() =>
  notificationsTab.value === 'read'
    ? notifications.value.filter((item) => item.read)
    : notifications.value
)

function toggleNotificationsMenu() {
  notificationsMenuOpen.value = !notificationsMenuOpen.value
}

function dismissNotification(notificationId) {
  notifications.value = notifications.value.filter((item) => item.id !== notificationId)
}

const accountMenuOpen = ref(false)
const accountMenuRef = ref(null)

function toggleAccountMenu() {
  accountMenuOpen.value = !accountMenuOpen.value
}

function signOut() {
  accountMenuOpen.value = false
  emit('sign-out')
}

function handleClickOutside(event) {
  if (workspaceSwitcherRef.value && !workspaceSwitcherRef.value.contains(event.target)) {
    workspaceMenuOpen.value = false
  }
  if (notificationsMenuRef.value && !notificationsMenuRef.value.contains(event.target)) {
    notificationsMenuOpen.value = false
  }
  if (accountMenuRef.value && !accountMenuRef.value.contains(event.target)) {
    accountMenuOpen.value = false
  }
}

onMounted(() => document.addEventListener('click', handleClickOutside))
onBeforeUnmount(() => document.removeEventListener('click', handleClickOutside))
</script>

<template>
  <header class="topnav" :class="{ collapsed }">
    <div class="topnav-left">
      <button type="button" class="icon-button" aria-label="Toggle sidebar" @click="$emit('toggle-sidebar')">
        <img class="icon-sidebar-toggle" :src="iconSidebarToggle" alt="" />
      </button>
      <span class="divider" aria-hidden="true"></span>
      <div class="workspace-menu" ref="workspaceSwitcherRef">
        <button
          type="button"
          class="workspace-switcher"
          :class="{ open: workspaceMenuOpen }"
          aria-haspopup="true"
          :aria-expanded="workspaceMenuOpen"
          @click="toggleWorkspaceMenu"
        >
          <span>Super Admin</span>
          <img class="chevron" :src="iconChevronSort" alt="" />
        </button>
        <div v-if="workspaceMenuOpen" class="workspace-dropdown" role="menu">
          <button
            v-for="mod in modules"
            :key="mod.key"
            type="button"
            class="workspace-dropdown-item"
            :class="{ active: mod.key === activeModule }"
            role="menuitem"
            @click="selectModule(mod.key)"
          >
            {{ mod.title }}
          </button>
        </div>
      </div>
    </div>

    <div class="topnav-right">
      <div class="notifications-menu" ref="notificationsMenuRef">
        <button
          type="button"
          class="icon-button"
          aria-label="Notifications"
          aria-haspopup="true"
          :aria-expanded="notificationsMenuOpen"
          @click="toggleNotificationsMenu"
        >
          <img class="icon-bell" :src="iconBell" alt="" />
        </button>
        <div v-if="notificationsMenuOpen" class="notifications-dropdown" role="menu">
          <div class="notifications-header">
            <p class="notifications-title">Notification</p>
            <div class="notifications-tabs">
              <button
                type="button"
                class="notifications-tab"
                :class="{ active: notificationsTab === 'all' }"
                @click="notificationsTab = 'all'"
              >
                <span>All</span>
                <span class="notifications-badge">{{ unreadCount }}</span>
              </button>
              <button
                type="button"
                class="notifications-tab"
                :class="{ active: notificationsTab === 'read' }"
                @click="notificationsTab = 'read'"
              >
                <span>Read</span>
              </button>
            </div>
          </div>

          <div class="notifications-list">
            <div
              v-for="item in visibleNotifications"
              :key="item.id"
              class="notification-card"
              :class="{ unread: !item.read }"
            >
              <img class="notification-avatar" :src="imgAvatar" alt="" />
              <div class="notification-body">
                <p class="notification-category">{{ item.category }}</p>
                <p class="notification-message">{{ item.message }}</p>
                <div class="notification-time">
                  <img class="icon-clock" :src="iconClock" alt="" />
                  <span>{{ item.time }}</span>
                </div>
              </div>
              <button
                type="button"
                class="notification-dismiss"
                aria-label="Dismiss notification"
                @click="dismissNotification(item.id)"
              >
                <img class="icon-trash" :src="iconTrash" alt="" />
              </button>
            </div>
          </div>

          <div class="notifications-footer">
            <button type="button" class="view-all-button" @click="$emit('open-notifications')">
              <span>View All Notification</span>
              <img class="icon-arrow-right" :src="iconArrowRight" alt="" />
            </button>
          </div>
        </div>
      </div>
      <div class="account-menu" ref="accountMenuRef">
        <button
          type="button"
          class="avatar-button"
          aria-label="Account"
          aria-haspopup="true"
          :aria-expanded="accountMenuOpen"
          @click="toggleAccountMenu"
        >
          <img class="avatar" :src="imgAvatar" alt="" />
        </button>
        <div v-if="accountMenuOpen" class="account-dropdown" role="menu">
          <div class="account-dropdown-header">
            <img class="account-avatar" :src="imgAvatar" alt="" />
            <div class="account-identity">
              <p class="account-name">Izzthdr</p>
              <p class="account-email">Izzthdr@Dynectus.com</p>
            </div>
          </div>
          <button type="button" class="account-dropdown-item" role="menuitem">
            <img class="icon-user" :src="iconUser" alt="" />
            <span>Account</span>
          </button>
          <button type="button" class="account-dropdown-item" role="menuitem">
            <img class="icon-question-circle" :src="iconQuestionCircle" alt="" />
            <span>Help center</span>
          </button>
          <span class="account-dropdown-separator" aria-hidden="true"></span>
          <button type="button" class="account-dropdown-item danger" role="menuitem" @click="signOut">
            <img class="icon-sign-out" :src="iconSignOut" alt="" />
            <span>Sign out</span>
          </button>
          <div class="account-dropdown-footer">© 2026 MCDynect</div>
        </div>
      </div>
    </div>
  </header>
</template>

<style scoped>
/* Sidebar is position: fixed and topnav needs to stay pinned to the
   viewport too (position: sticky inside a flex column is unreliable in
   Safari), so topnav is fixed as well and tracks the sidebar's width. */
.topnav {
  position: fixed;
  top: 0;
  left: 251px;
  right: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 66px;
  padding: 18px 32px 18px 24px;
  background: #ffffff;
  border-bottom: 1px solid #e5e7eb;
  flex-shrink: 0;
  transition: left 220ms ease;
}
.topnav.collapsed {
  left: 72px;
}

.topnav-left,
.topnav-right {
  display: flex;
  align-items: center;
  gap: 16px;
}

.icon-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  padding: 0;
  border: none;
  border-radius: 12px;
  background: transparent;
  cursor: pointer;
  transition: background 0.15s ease;
}
.icon-button:hover {
  background: rgba(249, 223, 227, 0.5);
}
/* Icons keep their exported Figma bounding box (not the 20px container
   square) so the vector's own aspect ratio isn't stretched. */
.icon-sidebar-toggle {
  width: 15px;
  height: 15px;
}
.icon-bell {
  width: 13.33px;
  height: 16.67px;
}

.divider {
  width: 1px;
  height: 24px;
  background: #e5e7eb;
}

.workspace-menu {
  position: relative;
}

.workspace-switcher {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: #4a5565;
  cursor: pointer;
}
.workspace-switcher .chevron {
  width: 6.67px;
  height: 10.67px;
  transition: transform 0.15s ease;
}
.workspace-switcher.open .chevron {
  transform: rotate(180deg);
}

.workspace-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 180px;
  padding: 8px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 10px 15px -3px rgba(29, 41, 61, 0.1), 0px 4px 6px -4px rgba(29, 41, 61, 0.1);
}

.workspace-dropdown-item {
  display: flex;
  align-items: center;
  width: 100%;
  padding: 8px;
  border: none;
  border-radius: 8px;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  line-height: 14px;
  color: #4a5565;
  text-align: left;
  white-space: nowrap;
  cursor: pointer;
  transition: background 0.15s ease;
}
.workspace-dropdown-item:hover {
  background: rgba(249, 223, 227, 0.5);
}
.workspace-dropdown-item.active {
  background: #f9dfe3;
  color: #911b2d;
}
.workspace-dropdown-item.active:hover {
  background: #f9dfe3;
}

.notifications-menu {
  position: relative;
}

.notifications-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  width: 348px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0px 20px 25px -5px rgba(29, 41, 61, 0.1), 0px 8px 10px -6px rgba(29, 41, 61, 0.1);
}

.notifications-header {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 16px 16px 0;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-bottom: none;
  border-radius: 12px 12px 0 0;
}

.notifications-title {
  font-size: 18px;
  font-weight: 600;
  line-height: 28px;
  color: #101828;
}

.notifications-tabs {
  display: flex;
  align-items: center;
  gap: 5px;
}

.notifications-tab {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 0 10px 16px 0;
  border: none;
  border-bottom: 1px solid transparent;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: #4a5565;
  cursor: pointer;
}
.notifications-tab.active {
  border-bottom-color: #1e2939;
  color: #101828;
}

.notifications-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  padding: 2px 6px;
  background: #BD2841;
  border-radius: 9999px;
  font-size: 12px;
  font-weight: 500;
  line-height: 16px;
  color: #ffffff;
}

.notifications-list {
  display: flex;
  flex-direction: column;
  max-height: 360px;
  overflow-y: auto;
}

.notification-card {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 16px;
  background: #ffffff;
  border-left: 1px solid #f3f4f6;
  border-right: 1px solid #f3f4f6;
  border-bottom: 1px solid #f3f4f6;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
}
.notification-card.unread {
  background: #FFF7F8;
}

.notification-avatar {
  flex-shrink: 0;
  width: 28px;
  height: 28px;
  border-radius: 100px;
  object-fit: cover;
}

.notification-body {
  display: flex;
  flex: 1 0 0;
  flex-direction: column;
  gap: 6px;
  min-width: 0;
}

.notification-category {
  font-size: 16px;
  font-weight: 500;
  line-height: 24px;
  color: #101828;
}

.notification-message {
  font-size: 16px;
  font-weight: 400;
  line-height: 24px;
  color: #4a5565;
}

.notification-time {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  font-weight: 400;
  color: #4a5565;
}
.notification-time .icon-clock {
  width: 14px;
  height: 14px;
}

.notification-dismiss {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  padding: 0;
  border: none;
  border-radius: 12px;
  background: transparent;
  cursor: pointer;
  transition: background 0.15s ease;
}
.notification-dismiss:hover {
  background: rgba(249, 223, 227, 0.5);
}
.notification-dismiss .icon-trash {
  width: 10.67px;
  height: 13.33px;
}

.notifications-footer {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px 16px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-top: none;
  border-radius: 0 0 12px 12px;
}

.view-all-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 6px 12px;
  border: none;
  border-radius: 12px;
  background: #911b2d;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 12px;
  font-weight: 500;
  line-height: 20px;
  color: #ffffff;
  cursor: pointer;
}
.view-all-button .icon-arrow-right {
  width: 14px;
  height: 14px;
}

.avatar-button {
  padding: 0;
  border: none;
  background: transparent;
  cursor: pointer;
  border-radius: 100px;
}
.avatar {
  display: block;
  width: 32px;
  height: 32px;
  border-radius: 100px;
  object-fit: cover;
}

.account-menu {
  position: relative;
}

.account-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  gap: 6px;
  width: 288px;
  padding: 8px 8px 0;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 10px 15px -3px rgba(29, 41, 61, 0.1), 0px 4px 6px -4px rgba(29, 41, 61, 0.1);
  overflow: clip;
}

.account-dropdown-header {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 10px;
  background: #f9fafb;
  border-radius: 8px;
}

.account-avatar {
  flex-shrink: 0;
  width: 32px;
  height: 32px;
  border-radius: 100px;
  object-fit: cover;
}

.account-identity {
  display: flex;
  flex: 1 0 0;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
}

.account-name {
  font-size: 14px;
  font-weight: 500;
  line-height: 14px;
  color: #101828;
}

.account-email {
  font-size: 14px;
  font-weight: 400;
  line-height: 20px;
  color: #4a5565;
}

.account-dropdown-item {
  display: flex;
  align-items: center;
  gap: 6px;
  width: 100%;
  padding: 8px;
  border: none;
  border-radius: 8px;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  line-height: 14px;
  color: #4a5565;
  text-align: left;
  white-space: nowrap;
  cursor: pointer;
  transition: background 0.15s ease;
}
.account-dropdown-item:hover {
  background: rgba(249, 223, 227, 0.5);
}
.account-dropdown-item.danger {
  color: #c70036;
}
.account-dropdown-item .icon-user,
.account-dropdown-item .icon-question-circle,
.account-dropdown-item .icon-sign-out {
  flex-shrink: 0;
  width: 16px;
  height: 16px;
}

.account-dropdown-separator {
  display: block;
  height: 1px;
  margin: 4px 0;
  background: #e5e7eb;
}

.account-dropdown-footer {
  margin: 0 -8px;
  padding: 8px 16px;
  background: #f9fafb;
  font-size: 12px;
  font-weight: 500;
  line-height: 24px;
  color: #99a1af;
  text-align: center;
}
</style>
