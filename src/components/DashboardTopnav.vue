<script setup>
/**
 * DashboardTopnav.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1232-3424
 * Workspace dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-8647
 * Notification dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-7280
 * Account dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-8006
 * Ask Dyno button: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1232-3397
 *
 * Top bar for the SuperAdmin dashboard: sidebar toggle, workspace switcher,
 * notifications, avatar, and the "Ask Dyno" button.
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
  showWorkspaceSwitcher: {
    type: Boolean,
    default: true,
  },
})

const emit = defineEmits(['toggle-sidebar', 'switch-workspace', 'open-notifications', 'sign-out', 'ask-dyno'])

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
      <template v-if="showWorkspaceSwitcher">
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
      </template>
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
            <span class="account-dropdown-icon">
              <img class="icon-user" :src="iconUser" alt="" />
            </span>
            <span>Account</span>
          </button>
          <button type="button" class="account-dropdown-item" role="menuitem">
            <span class="account-dropdown-icon">
              <img class="icon-question-circle" :src="iconQuestionCircle" alt="" />
            </span>
            <span>Help center</span>
          </button>
          <span class="account-dropdown-separator" aria-hidden="true"></span>
          <button type="button" class="account-dropdown-item danger" role="menuitem" @click="signOut">
            <span class="account-dropdown-icon">
              <img class="icon-sign-out" :src="iconSignOut" alt="" />
            </span>
            <span>Sign out</span>
          </button>
          <div class="account-dropdown-footer">© 2026 MCDynect</div>
        </div>
      </div>
      <button type="button" class="ask-dyno-button" @click="$emit('ask-dyno')">
        <span>Ask Dyno</span>
        <svg class="icon-headset" viewBox="0 0 13.3333 13.3333" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
          <path
            fill-rule="evenodd"
            clip-rule="evenodd"
            d="M3.36684 1.36684C4.242 0.491665 5.42899 0 6.66667 0C7.90434 0 9.09133 0.491665 9.9665 1.36684C10.8417 2.242 11.3333 3.42899 11.3333 4.66667C11.8638 4.66667 12.3725 4.87738 12.7475 5.25245C13.1226 5.62752 13.3333 6.13623 13.3333 6.66667V8C13.3333 8.53043 13.1226 9.03914 12.7475 9.41421C12.3725 9.78929 11.8638 10 11.3333 10H11.2758C11.2341 10.2198 11.1699 10.4354 11.0838 10.6431C10.9191 11.0408 10.6776 11.4022 10.3732 11.7066C10.0688 12.011 9.70747 12.2524 9.30977 12.4171C8.91206 12.5819 8.48581 12.6667 8.05533 12.6667H7.82162C7.59108 13.0652 7.16019 13.3333 6.66667 13.3333H6C5.26362 13.3333 4.66667 12.7364 4.66667 12V11.3333C4.66667 10.597 5.26362 10 6 10H6.66667C7.40305 10 8 10.597 8 11.3333H8.05533C8.31071 11.3333 8.56359 11.283 8.79952 11.1853C9.03546 11.0876 9.24984 10.9443 9.43042 10.7638C9.611 10.5832 9.75424 10.3688 9.85197 10.1329C9.9497 9.89692 10 9.64404 10 9.38867V4.66667C10 3.78261 9.64881 2.93477 9.02369 2.30964C8.39857 1.68452 7.55072 1.33333 6.66667 1.33333C5.78261 1.33333 4.93477 1.68452 4.30964 2.30964C3.68452 2.93477 3.33333 3.78261 3.33333 4.66667V9.33333C3.33333 9.70152 3.03486 10 2.66667 10H2C1.46957 10 0.960859 9.78929 0.585786 9.41421C0.210714 9.03914 0 8.53043 0 8V6.66667C0 6.13623 0.210714 5.62753 0.585786 5.25245C0.960859 4.87738 1.46957 4.66667 2 4.66667C2 3.42899 2.49167 2.242 3.36684 1.36684ZM11.3333 8.66667C11.5101 8.66667 11.6797 8.59643 11.8047 8.4714C11.9298 8.34638 12 8.17681 12 8V6.66667C12 6.48986 11.9298 6.32029 11.8047 6.19526C11.6797 6.07024 11.5101 6 11.3333 6V8.66667ZM6.6048 2.0027C6.95633 1.99444 7.30602 2.05578 7.63375 2.1832C7.96148 2.31061 8.26077 2.50158 8.5144 2.74512C8.77998 3.00013 8.78855 3.42216 8.53354 3.68774C8.27853 3.95332 7.85651 3.96189 7.59093 3.70688C7.46411 3.58511 7.31447 3.48962 7.1506 3.42591C6.98674 3.36221 6.81189 3.33154 6.63613 3.33567C6.46036 3.3398 6.28715 3.37865 6.12646 3.44998C5.96577 3.52132 5.82077 3.62373 5.69982 3.75132C5.44651 4.01853 5.02455 4.02979 4.75734 3.77648C4.49014 3.52318 4.47887 3.10122 4.73218 2.83401C4.97409 2.57883 5.26409 2.37401 5.58547 2.23134C5.90685 2.08866 6.25327 2.01096 6.6048 2.0027ZM2 6C1.82319 6 1.65362 6.07024 1.5286 6.19526C1.40357 6.32029 1.33333 6.48986 1.33333 6.66667V8C1.33333 8.17681 1.40357 8.34638 1.5286 8.4714C1.65362 8.59643 1.82319 8.66667 2 8.66667V6ZM6.66667 11.3333H6V12H6.66667V11.3333Z"
            fill="currentColor"
          />
        </svg>
      </button>
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
  left: 64px;
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
  background: #fae8ea;
  color: #b13444;
}
.workspace-dropdown-item.active:hover {
  background: #fae8ea;
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
  background: #b13444;
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
  display: flex;
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
/* 16x16 alignment slot so Account/Help center/Sign out text stays flush,
   regardless of each icon's own (non-square) exported aspect ratio. */
.account-dropdown-icon {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: 16px;
  height: 16px;
}
.account-dropdown-icon .icon-user {
  width: 8px;
  height: 10.67px;
}
.account-dropdown-icon .icon-question-circle {
  width: 13.33px;
  height: 13.33px;
}
.account-dropdown-icon .icon-sign-out {
  width: 12px;
  height: 12px;
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

/* Liquid-fill hover, ported from animate-ui's LiquidButton primitive
   (https://animate-ui.com/docs/primitives/buttons/liquid): a brand-color
   fill rises from a hairline at the bottom to a full fill, then the
   label/icon invert to white once the fill lands. */
.ask-dyno-button {
  position: relative;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  border: 1px solid #b13444;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: #b13444;
  cursor: pointer;
  background-color: #f9fafb;
  background-image: linear-gradient(#b13444 0 0);
  background-repeat: no-repeat;
  background-position: 0% 100%;
  background-size: 100% 3px;
  transform: scale(1);
  transition:
    background-size 0.3s ease,
    border-color 0.3s ease,
    color 0.3s ease 0.3s,
    transform 0.15s ease;
}
.ask-dyno-button:hover {
  background-size: 100% 100%;
  border-color: #b13444;
  color: #ffffff;
  transform: scale(1.05);
}
.ask-dyno-button:active {
  transform: scale(0.95);
}
.ask-dyno-button .icon-headset {
  width: 13.33px;
  height: 13.33px;
  flex-shrink: 0;
}
</style>
