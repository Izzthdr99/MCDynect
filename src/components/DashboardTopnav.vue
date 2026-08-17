<script setup>
/**
 * DashboardTopnav.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1232-3424
 * Workspace dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-8647
 * Notification dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-7280
 * Account dropdown: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1284-8006
 * Ask Dyno button: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1232-3397
 * Ask Dyno menu (have a ticket): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1459-8621
 * Ask Dyno menu (no ticket): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1463-8847
 * Ticket form (opened from any Ask Dyno menu item): https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1466-9310
 * Ticket submitted panel: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1472-9672
 * Draft saved panel: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1475-9804
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
import iconArrowRightMuted from '../assets/dashboard/topnav/icon-arrow-right-muted.svg'
import iconUser from '../assets/dashboard/topnav/icon-user.svg'
import iconQuestionCircle from '../assets/dashboard/topnav/icon-question-circle.svg'
import iconSignOut from '../assets/dashboard/topnav/icon-sign-out.svg'
import iconClose from '../assets/dashboard/topnav/icon-close.svg'
import iconChevronDown from '../assets/dashboard/topnav/icon-chevron-down.svg'
import iconUpload from '../assets/dashboard/topnav/icon-upload.svg'
import iconCheckCircle from '../assets/dashboard/topnav/icon-check-circle.svg'
import iconWarningTriangle from '../assets/dashboard/topnav/icon-warning-triangle.svg'
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

const emit = defineEmits([
  'toggle-sidebar',
  'switch-workspace',
  'open-notifications',
  'sign-out',
  'view-ticket',
  'ask-dyno-item',
  'submit-ticket',
  'save-draft-ticket',
])

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

const askDynoMenuOpen = ref(false)
const askDynoMenuRef = ref(null)

// Mock: swap for the user's real open-ticket count once support tickets are wired up.
const openTicketCount = ref(1)
const hasOpenTicket = computed(() => openTicketCount.value > 0)

const askDynoItems = [
  {
    key: 'bug',
    label: 'Report a bug',
    iconClass: 'icon-bug',
    iconViewBox: '0 0 13.3333 11.3333',
    iconPath:
      'M6.66667 0C7.37391 0 8.05199 0.281153 8.55208 0.78125C9.05218 1.28135 9.33333 1.95942 9.33333 2.66667C9.43359 2.66667 9.53051 2.69004 9.61849 2.73177L11.6406 1.4388C11.9507 1.24036 12.3627 1.33063 12.5612 1.64062C12.7596 1.95069 12.6694 2.36269 12.3594 2.5612L10.349 3.84766C10.53 4.30892 10.6379 4.79788 10.6654 5.29687C10.666 5.30902 10.6667 5.32117 10.6667 5.33333H12.6667C13.0349 5.33333 13.3333 5.63181 13.3333 6C13.3333 6.36819 13.0349 6.66667 12.6667 6.66667H10.6667V7.33333C10.6667 7.67897 10.6208 8.01968 10.5345 8.34831L12.3418 9.42773C12.6577 9.61649 12.7615 10.0258 12.5729 10.3418C12.3842 10.6579 11.9743 10.7608 11.6582 10.5723L9.98047 9.57031C9.83841 9.78069 9.67689 9.97936 9.49479 10.1615C8.74465 10.9116 7.72753 11.3333 6.66667 11.3333C5.6058 11.3333 4.58869 10.9116 3.83854 10.1615C3.65635 9.97927 3.49432 9.7808 3.35221 9.57031L1.67513 10.5723C1.35903 10.761 0.949835 10.6578 0.761068 10.3418C0.572351 10.0257 0.675485 9.6165 0.991536 9.42773L2.79818 8.34831C2.7119 8.01973 2.66667 7.67891 2.66667 7.33333V6.66667H0.666667C0.298477 6.66667 0 6.36819 0 6C0 5.63181 0.298477 5.33333 0.666667 5.33333H2.66667C2.66667 5.32117 2.6673 5.30902 2.66797 5.29687C2.69548 4.79796 2.8027 4.30885 2.98372 3.84766L0.973958 2.5612C0.663963 2.36269 0.573697 1.95069 0.772135 1.64062C0.970646 1.33063 1.38264 1.24036 1.69271 1.4388L3.71419 2.73177C3.80231 2.68988 3.89954 2.66667 4 2.66667C4 1.95942 4.28115 1.28135 4.78125 0.78125C5.28135 0.281153 5.95942 0 6.66667 0ZM4.38021 4C4.15941 4.41886 4.02829 4.8803 4 5.35547V7.33333C4 8.04058 4.28115 8.71865 4.78125 9.21875C5.28135 9.71885 5.95942 10 6.66667 10C7.37391 10 8.05199 9.71885 8.55208 9.21875C9.05218 8.71865 9.33333 8.04058 9.33333 7.33333V5.35547C9.30505 4.8803 9.17393 4.41886 8.95312 4H7.33333V7.33333C7.33333 7.70152 7.03486 8 6.66667 8C6.29848 8 6 7.70152 6 7.33333V4H4.38021ZM6.66667 1.33333C6.31304 1.33333 5.97401 1.47391 5.72396 1.72396C5.47391 1.97401 5.33333 2.31304 5.33333 2.66667H8C8 2.31304 7.85942 1.97401 7.60937 1.72396C7.35933 1.47391 7.02029 1.33333 6.66667 1.33333Z',
  },
  {
    key: 'change-request',
    label: 'Request for change (CR)',
    iconClass: 'icon-clipboard',
    iconViewBox: '0 0 10.6667 13.3333',
    iconPath:
      'M3.33333 0.666667C3.33333 0.298477 3.63181 0 4 0H6.66667C7.03486 0 7.33333 0.298477 7.33333 0.666667H9.33333C10.0697 0.666667 10.6667 1.26362 10.6667 2V12C10.6667 12.7364 10.0697 13.3333 9.33333 13.3333H1.33333C0.596954 13.3333 0 12.7364 0 12V2C0 1.26362 0.596954 0.666667 1.33333 0.666667H3.33333ZM3.33333 2H1.33333V12H9.33333V2H7.33333V2.66667C7.70152 2.66667 8 2.96514 8 3.33333C8 3.70152 7.70152 4 7.33333 4H3.33333C2.96514 4 2.66667 3.70152 2.66667 3.33333C2.66667 2.96514 2.96514 2.66667 3.33333 2.66667V2ZM4.66667 2.66667H6V1.33333H4.66667V2.66667ZM2.66667 6.66667C2.66667 6.29848 2.96514 6 3.33333 6H7.33333C7.70152 6 8 6.29848 8 6.66667C8 7.03486 7.70152 7.33333 7.33333 7.33333H3.33333C2.96514 7.33333 2.66667 7.03486 2.66667 6.66667ZM2.66667 9.33333C2.66667 8.96514 2.96514 8.66667 3.33333 8.66667H7.33333C7.70152 8.66667 8 8.96514 8 9.33333C8 9.70152 7.70152 10 7.33333 10H3.33333C2.96514 10 2.66667 9.70152 2.66667 9.33333Z',
  },
  {
    key: 'improvement',
    label: 'Suggest an improvement',
    iconClass: 'icon-lightbulb',
    iconViewBox: '0 0 9.33333 13.3333',
    iconPath:
      'M1.36684 1.36684C2.242 0.491665 3.42899 0 4.66667 0C5.90434 0 7.09133 0.491665 7.9665 1.36684C8.84167 2.242 9.33333 3.42899 9.33333 4.66667C9.33333 6.27153 8.53509 7.24904 7.87608 8.05603C7.86728 8.0668 7.85852 8.07754 7.84977 8.08825C7.18031 8.90835 6.66667 9.55962 6.66667 10.6667C6.66667 11.0349 6.36819 11.3333 6 11.3333H3.33333C2.96514 11.3333 2.66667 11.0349 2.66667 10.6667C2.66667 9.59183 2.15293 8.92502 1.47942 8.08313C1.46954 8.07078 1.45963 8.0584 1.44969 8.04597C0.795244 7.22828 0 6.23467 0 4.66667C0 3.42899 0.491665 2.242 1.36684 1.36684ZM4.66667 1.33333C3.78261 1.33333 2.93477 1.68452 2.30964 2.30964C1.68452 2.93477 1.33333 3.78261 1.33333 4.66667C1.33333 5.7415 1.84707 6.40832 2.52058 7.2502C2.53046 7.26255 2.54037 7.27493 2.55031 7.28736C3.10682 7.98269 3.76515 8.80523 3.94971 10H5.38182C5.56433 8.77892 6.22817 7.96602 6.79059 7.2773C6.79938 7.26653 6.80815 7.25579 6.81689 7.24508C7.48636 6.42499 8 5.77372 8 4.66667C8 3.78261 7.64881 2.93477 7.02369 2.30964C6.39857 1.68452 5.55072 1.33333 4.66667 1.33333ZM4.66667 3.33333C4.31304 3.33333 3.97391 3.47381 3.72386 3.72386C3.47381 3.97391 3.33333 4.31304 3.33333 4.66667C3.33333 5.03486 3.03486 5.33333 2.66667 5.33333C2.29848 5.33333 2 5.03486 2 4.66667C2 3.95942 2.28095 3.28115 2.78105 2.78105C3.28115 2.28095 3.95942 2 4.66667 2C5.03486 2 5.33333 2.29848 5.33333 2.66667C5.33333 3.03486 5.03486 3.33333 4.66667 3.33333ZM2.66667 12.6667C2.66667 12.2985 2.96514 12 3.33333 12H6C6.36819 12 6.66667 12.2985 6.66667 12.6667C6.66667 13.0349 6.36819 13.3333 6 13.3333H3.33333C2.96514 13.3333 2.66667 13.0349 2.66667 12.6667Z',
  },
  {
    key: 'help',
    label: 'Need help / How to',
    iconClass: 'icon-life-saver',
    iconViewBox: '0 0 13.3363 13.3363',
    iconPath:
      'M3.29859 0.919157C4.31731 0.318957 5.48085 0 6.66947 0C7.85756 0 9.0206 0.318673 10.039 0.918354C10.1147 0.8716 10.1942 0.830874 10.2768 0.796654C10.479 0.71283 10.6958 0.669685 10.9148 0.669685C11.1338 0.669685 11.3506 0.71283 11.5529 0.796654C11.7551 0.880451 11.9388 1.00326 12.0935 1.15807C12.0936 1.15812 12.0935 1.15801 12.0935 1.15807L12.1781 1.24258C12.3329 1.39736 12.4558 1.58114 12.5396 1.78342C12.6234 1.9857 12.6666 2.20251 12.6666 2.42147C12.6666 2.64043 12.6234 2.85725 12.5396 3.05952C12.5052 3.14257 12.4642 3.2225 12.4171 3.29859C13.0173 4.31731 13.3363 5.48085 13.3363 6.66947C13.3363 7.85756 13.0176 9.0206 12.4179 10.039C12.4647 10.1147 12.5054 10.1942 12.5396 10.2768C12.6234 10.479 12.6666 10.6958 12.6666 10.9148C12.6666 11.1338 12.6234 11.3506 12.5396 11.5529C12.4558 11.7551 12.3329 11.9389 12.1781 12.0937L12.1763 12.0954L12.0937 12.1774C12.0933 12.1778 12.093 12.1781 12.0926 12.1785C11.938 12.3328 11.7547 12.4553 11.5529 12.539C11.3506 12.6228 11.1338 12.6659 10.9148 12.6659C10.6958 12.6659 10.479 12.6228 10.2768 12.539C10.1939 12.5046 10.1142 12.4637 10.0382 12.4168C9.01939 13.0172 7.85564 13.3363 6.6668 13.3363C5.4785 13.3363 4.31526 13.0175 3.29674 12.4176C3.22121 12.4642 3.1419 12.5048 3.05952 12.539C2.85725 12.6228 2.64043 12.6659 2.42147 12.6659C2.20251 12.6659 1.9857 12.6228 1.78342 12.539C1.58165 12.4553 1.39828 12.3329 1.24375 12.1786C1.24336 12.1782 1.24297 12.1778 1.24258 12.1774L1.15993 12.0954L1.15822 12.0937C1.00334 11.9389 0.880479 11.7551 0.796654 11.5529C0.71283 11.3506 0.669685 11.1338 0.669685 10.9148C0.669685 10.6958 0.71283 10.479 0.796654 10.2768C0.83107 10.1937 0.872067 10.1138 0.919158 10.0377C0.318958 9.01897 0 7.85542 0 6.6668C0 5.47872 0.318673 4.31568 0.918352 3.29729C0.871599 3.22159 0.830874 3.1421 0.796654 3.05952C0.71283 2.85724 0.669685 2.64043 0.669685 2.42147C0.669685 2.20251 0.71283 1.9857 0.796654 1.78342C0.880452 1.58121 1.00326 1.39748 1.15807 1.24273L1.24258 1.15822C1.24263 1.15817 1.24253 1.15827 1.24258 1.15822C1.39733 1.00341 1.58121 0.880452 1.78342 0.796654C1.9857 0.71283 2.20251 0.669685 2.42147 0.669685C2.64043 0.669685 2.85724 0.71283 3.05952 0.796654C3.14257 0.83107 3.2225 0.872066 3.29859 0.919157ZM4.60548 4.04946L2.65734 2.10081C2.62638 2.06984 2.58954 2.04518 2.54908 2.02841C2.50863 2.01165 2.46526 2.00302 2.42147 2.00302C2.37768 2.00302 2.33432 2.01165 2.29386 2.02841C2.2534 2.04518 2.21665 2.06975 2.18569 2.10072L2.10088 2.18554C2.0699 2.2165 2.04518 2.2534 2.02841 2.29386C2.01165 2.33432 2.00302 2.37768 2.00302 2.42147C2.00302 2.46526 2.01165 2.50863 2.02841 2.54908C2.04518 2.58954 2.06975 2.62629 2.10072 2.65725L2.19849 2.75499C2.2044 2.76048 2.21022 2.76609 2.21595 2.77181L4.05041 4.60427C4.21303 4.3983 4.39943 4.21198 4.60548 4.04946ZM1.87961 4.32156L3.43047 5.87202C3.36796 6.12713 3.3348 6.39376 3.3348 6.66814C3.3348 6.94177 3.36778 7.2077 3.42996 7.46217L1.87977 9.01237C1.52187 8.28592 1.33333 7.48377 1.33333 6.6668C1.33333 5.84996 1.52181 5.04793 1.87961 4.32156ZM4.04946 8.7308L2.10081 10.6789C2.06984 10.7099 2.04518 10.7467 2.02841 10.7872C2.01165 10.8277 2.00302 10.871 2.00302 10.9148C2.00302 10.9586 2.01165 11.002 2.02841 11.0424C2.04507 11.0826 2.06943 11.1191 2.10012 11.15C2.10032 11.1502 2.10052 11.1504 2.10072 11.1506L2.18367 11.2329L2.18569 11.2349C2.21665 11.2659 2.2534 11.2904 2.29386 11.3072C2.33432 11.324 2.37768 11.3326 2.42147 11.3326C2.46526 11.3326 2.50863 11.324 2.54908 11.3072C2.58954 11.2904 2.62629 11.2659 2.65725 11.2349L4.60427 9.28586C4.3983 9.12325 4.21198 8.93685 4.04946 8.7308ZM4.32124 11.4565C5.04769 11.8144 5.84984 12.0029 6.6668 12.0029C7.48377 12.0029 8.28592 11.8144 9.01237 11.4565L7.46217 9.90632C7.2077 9.9685 6.94177 10.0015 6.66814 10.0015C6.39374 10.0015 6.12708 9.96832 5.87196 9.90579L4.32124 11.4565ZM6.02696 8.56315C6.22824 8.63123 6.44388 8.66814 6.66814 8.66814C6.91692 8.66814 7.15509 8.62271 7.37482 8.53971C7.38338 8.53585 7.39204 8.53216 7.4008 8.52865C7.65459 8.42694 7.88482 8.27428 8.07728 8.0801C8.26975 7.88591 8.42036 7.65433 8.5198 7.39965C8.53207 7.36823 8.54659 7.33806 8.56315 7.30931C8.63123 7.10804 8.66814 6.8924 8.66814 6.66814C8.66814 6.41935 8.62271 6.18118 8.53971 5.96146C8.53586 5.95292 8.53218 5.94429 8.52868 5.93555C8.42692 5.68172 8.27418 5.45146 8.07991 5.25899C7.88563 5.06652 7.65395 4.91594 7.39918 4.81655C7.36735 4.80413 7.3368 4.78941 7.3077 4.77258C7.10689 4.70485 6.8918 4.66814 6.66814 4.66814C6.41936 4.66814 6.18118 4.71356 5.96146 4.79657C5.9529 4.80043 5.94424 4.80411 5.93547 4.80763C5.68169 4.90933 5.45146 5.06199 5.25899 5.25618C5.06653 5.45036 4.91592 5.68195 4.81647 5.93663C4.80421 5.96804 4.78969 5.99821 4.77313 6.02696C4.70505 6.22824 4.66814 6.44388 4.66814 6.66814C4.66814 6.91692 4.71356 7.15509 4.79657 7.37482C4.80043 7.38338 4.80411 7.39204 4.80763 7.4008C4.90933 7.65459 5.06199 7.88482 5.25618 8.07728C5.45036 8.26975 5.68195 8.42036 5.93663 8.5198C5.96804 8.53207 5.99821 8.54659 6.02696 8.56315ZM5.8741 3.42996C6.12858 3.36778 6.39451 3.3348 6.66814 3.3348C6.94235 3.3348 7.20882 3.36791 7.46378 3.43035L9.01459 1.87955C8.28826 1.52179 7.48627 1.33333 6.66947 1.33333C5.8525 1.33333 5.05036 1.52187 4.32391 1.87977L5.8741 3.42996ZM9.90632 5.8741C9.9685 6.12858 10.0015 6.39451 10.0015 6.66814C10.0015 6.94252 9.96832 7.20915 9.90581 7.46425L11.4567 9.01471C11.8145 8.28835 12.0029 7.48632 12.0029 6.66947C12.0029 5.8525 11.8144 5.05036 11.4565 4.32391L9.90632 5.8741ZM9.28681 4.60548L11.2355 2.65734C11.2664 2.62638 11.2911 2.58954 11.3079 2.54908C11.3246 2.50863 11.3333 2.46526 11.3333 2.42147C11.3333 2.37768 11.3246 2.33432 11.3079 2.29386C11.2911 2.2534 11.2665 2.21665 11.2356 2.18569L11.1507 2.10088C11.1198 2.0699 11.0829 2.04518 11.0424 2.02841C11.002 2.01165 10.9586 2.00302 10.9148 2.00302C10.871 2.00302 10.8277 2.01165 10.7872 2.02841C10.7467 2.04518 10.71 2.06975 10.679 2.10072L10.5816 2.19815C10.5759 2.20429 10.5701 2.21033 10.5641 2.21627L8.73061 4.04931C8.93713 4.21219 9.12393 4.39897 9.28681 4.60548ZM8.73089 9.28674L10.6789 11.2347C10.7098 11.2657 10.7467 11.2904 10.7872 11.3072C10.8277 11.324 10.871 11.3326 10.9148 11.3326C10.9586 11.3326 11.002 11.324 11.0424 11.3072C11.0829 11.2904 11.1196 11.2659 11.1506 11.2349L11.1526 11.2329L11.2356 11.1506C11.2358 11.1504 11.236 11.1502 11.2362 11.15C11.2668 11.1191 11.2912 11.0826 11.3079 11.0424C11.3246 11.002 11.3333 10.9586 11.3333 10.9148C11.3333 10.871 11.3246 10.8277 11.3079 10.7872C11.2911 10.7467 11.2665 10.71 11.2356 10.679L11.1378 10.5813C11.1318 10.5757 11.1259 10.57 11.1201 10.5642L9.28674 8.73089C9.12395 8.93727 8.93727 9.12395 8.73089 9.28674Z',
  },
  {
    key: 'access',
    label: 'Access / Permission Request / Issue',
    iconClass: 'icon-lock-open',
    iconViewBox: '0 0 12 12',
    iconPath:
      'M9.33333 1.33333C8.59695 1.33333 8 1.93029 8 2.66667V4.66667C8.73638 4.66667 9.33333 5.26362 9.33333 6V10.6667C9.33333 11.403 8.73638 12 8 12H1.33333C0.596954 12 0 11.403 0 10.6667V6C0 5.26362 0.596954 4.66667 1.33333 4.66667H6.66667V2.66667C6.66667 1.19391 7.86057 0 9.33333 0C10.8061 0 12 1.19391 12 2.66667V5.33333C12 5.70152 11.7015 6 11.3333 6C10.9651 6 10.6667 5.70152 10.6667 5.33333V2.66667C10.6667 1.93029 10.0697 1.33333 9.33333 1.33333ZM1.33333 6V10.6667H8V6H1.33333ZM4.66667 6.66667C5.03486 6.66667 5.33333 6.96514 5.33333 7.33333V9.33333C5.33333 9.70152 5.03486 10 4.66667 10C4.29848 10 4 9.70152 4 9.33333V7.33333C4 6.96514 4.29848 6.66667 4.66667 6.66667Z',
  },
  {
    key: 'other',
    label: 'Other',
    iconClass: 'icon-dots-vertical',
    iconViewBox: '0 0 2.00667 10',
    iconPath:
      'M0 1C0 0.447715 0.447715 0 1 0H1.00667C1.55895 0 2.00667 0.447715 2.00667 1C2.00667 1.55228 1.55895 2 1.00667 2H1C0.447715 2 0 1.55228 0 1ZM0 5C0 4.44772 0.447715 4 1 4H1.00667C1.55895 4 2.00667 4.44772 2.00667 5C2.00667 5.55229 1.55895 6 1.00667 6H1C0.447715 6 0 5.55229 0 5ZM0 9C0 8.44772 0.447715 8 1 8H1.00667C1.55895 8 2.00667 8.44772 2.00667 9C2.00667 9.55229 1.55895 10 1.00667 10H1C0.447715 10 0 9.55229 0 9Z',
  },
]

function toggleAskDynoMenu() {
  askDynoMenuOpen.value = !askDynoMenuOpen.value
}

// Every Ask Dyno item opens the same Ticket Form (Figma node 1466-9310),
// preselecting Subject to whichever item was clicked (e.g. "Report a bug").
function selectAskDynoItem(key) {
  const item = askDynoItems.find((entry) => entry.key === key)
  askDynoMenuOpen.value = false
  if (item) {
    openTicketForm(item)
  }
  emit('ask-dyno-item', key)
}

function viewTicket() {
  askDynoMenuOpen.value = false
  emit('view-ticket')
}

const ticketFormOpen = ref(false)
const ticketSubmitted = ref(false)
const submittedTicketNumber = ref('')
const draftSaved = ref(false)
const savedDraftId = ref('')
// Mock: swap for the real ticket/draft ID returned by the support API once it's wired up.
const nextTicketSequence = ref(2)
const priorityOptions = ['Low', 'Medium', 'High', 'Critical']
const MAX_UPLOAD_FILES = 5

function makeEmptyTicketForm() {
  return {
    subject: '',
    bugTitle: '',
    description: '',
    expectedResult: '',
    priority: '',
    phone: '',
  }
}

const ticketForm = ref(makeEmptyTicketForm())
const selectedFiles = ref([])
const fileInputRef = ref(null)

function openTicketForm(item) {
  ticketForm.value = makeEmptyTicketForm()
  ticketForm.value.subject = item.label
  selectedFiles.value = []
  ticketSubmitted.value = false
  draftSaved.value = false
  ticketFormOpen.value = true
}

function closeTicketForm() {
  ticketFormOpen.value = false
  ticketSubmitted.value = false
  draftSaved.value = false
  subjectMenuOpen.value = false
  priorityMenuOpen.value = false
}

const subjectMenuOpen = ref(false)
const subjectMenuRef = ref(null)

function toggleSubjectMenu() {
  subjectMenuOpen.value = !subjectMenuOpen.value
}

function selectSubject(label) {
  ticketForm.value.subject = label
  subjectMenuOpen.value = false
}

const priorityMenuOpen = ref(false)
const priorityMenuRef = ref(null)

function togglePriorityMenu() {
  priorityMenuOpen.value = !priorityMenuOpen.value
}

function selectPriority(option) {
  ticketForm.value.priority = option
  priorityMenuOpen.value = false
}

function triggerFileBrowse() {
  fileInputRef.value?.click()
}

function handleFileChange(event) {
  const incoming = Array.from(event.target.files ?? []).filter((file) => file.type.startsWith('image/'))
  const remainingSlots = MAX_UPLOAD_FILES - selectedFiles.value.length
  selectedFiles.value = [...selectedFiles.value, ...incoming.slice(0, remainingSlots)]
  // Reset so selecting the same file(s) again still fires a change event.
  event.target.value = ''
}

function removeSelectedFile(index) {
  selectedFiles.value = selectedFiles.value.filter((_, i) => i !== index)
}

function submitTicket() {
  const ticketNumber = `HD-2026-${String(nextTicketSequence.value).padStart(5, '0')}`
  nextTicketSequence.value += 1
  emit('submit-ticket', { ...ticketForm.value, files: selectedFiles.value, ticketNumber })
  submittedTicketNumber.value = ticketNumber
  ticketSubmitted.value = true
}

function viewSubmittedTicket() {
  closeTicketForm()
  emit('view-ticket')
}

function saveDraftTicket() {
  const draftId = `HD-2026-${String(nextTicketSequence.value).padStart(5, '0')}`
  nextTicketSequence.value += 1
  emit('save-draft-ticket', { ...ticketForm.value, files: selectedFiles.value, draftId })
  savedDraftId.value = draftId
  draftSaved.value = true
}

function editDraft() {
  draftSaved.value = false
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
  if (askDynoMenuRef.value && !askDynoMenuRef.value.contains(event.target)) {
    askDynoMenuOpen.value = false
  }
  if (subjectMenuRef.value && !subjectMenuRef.value.contains(event.target)) {
    subjectMenuOpen.value = false
  }
  if (priorityMenuRef.value && !priorityMenuRef.value.contains(event.target)) {
    priorityMenuOpen.value = false
  }
}

function handleKeydown(event) {
  if (event.key === 'Escape' && ticketFormOpen.value) {
    closeTicketForm()
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  document.addEventListener('keydown', handleKeydown)
})
onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
  document.removeEventListener('keydown', handleKeydown)
})
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
      <div class="ask-dyno-menu" ref="askDynoMenuRef">
        <button
          type="button"
          class="ask-dyno-button"
          :class="{ open: askDynoMenuOpen }"
          aria-haspopup="true"
          :aria-expanded="askDynoMenuOpen"
          @click="toggleAskDynoMenu"
        >
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
        <div v-if="askDynoMenuOpen" class="ask-dyno-dropdown" role="menu">
          <div v-if="hasOpenTicket" class="ask-dyno-cta">
            <div class="ask-dyno-cta-pattern" aria-hidden="true"></div>
            <div class="ask-dyno-cta-label">
              <p class="ask-dyno-cta-heading">You have an open ticket</p>
              <button type="button" class="view-ticket-button" @click="viewTicket">
                <span>View ticket</span>
                <img class="icon-arrow-right" :src="iconArrowRightMuted" alt="" />
              </button>
            </div>
            <div class="ask-dyno-cta-count">
              <p class="ask-dyno-cta-count-number">{{ openTicketCount }}</p>
              <p class="ask-dyno-cta-count-label">ticket{{ openTicketCount === 1 ? '' : 's' }}</p>
            </div>
          </div>
          <div class="ask-dyno-list">
            <template v-for="(item, index) in askDynoItems" :key="item.key">
              <button
                type="button"
                class="ask-dyno-item"
                role="menuitem"
                @click="selectAskDynoItem(item.key)"
              >
                <span class="ask-dyno-item-icon">
                  <svg :class="item.iconClass" :viewBox="item.iconViewBox" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                    <path fill-rule="evenodd" clip-rule="evenodd" :d="item.iconPath" fill="currentColor" />
                  </svg>
                </span>
                <span>{{ item.label }}</span>
              </button>
              <span v-if="index < askDynoItems.length - 1" class="ask-dyno-separator" aria-hidden="true"></span>
            </template>
          </div>
        </div>
      </div>
    </div>
  </header>

  <Teleport to="body">
    <div v-if="ticketFormOpen" class="ticket-form-overlay" @click="closeTicketForm">
      <div v-if="!ticketSubmitted && !draftSaved" class="ticket-form-modal" role="dialog" aria-modal="true" aria-label="Ticket Form" @click.stop>
        <div class="ticket-form-header">
          <div class="ticket-form-heading">
            <p class="ticket-form-title">Ticket Form</p>
            <p class="ticket-form-subtitle">Tell us what went wrong so we can help fix it quickly.</p>
          </div>
          <button type="button" class="icon-button" aria-label="Close ticket form" @click="closeTicketForm">
            <img class="icon-close" :src="iconClose" alt="" />
          </button>
        </div>

        <div class="ticket-form-body">
          <div class="ticket-form-field">
            <label class="ticket-form-label">Subject <span class="required">*</span></label>
            <div class="ticket-form-select" ref="subjectMenuRef">
              <button
                type="button"
                class="ticket-form-input select"
                :class="{ open: subjectMenuOpen }"
                aria-haspopup="listbox"
                :aria-expanded="subjectMenuOpen"
                @click="toggleSubjectMenu"
              >
                <span>{{ ticketForm.subject }}</span>
                <img class="icon-chevron-down" :src="iconChevronDown" alt="" />
              </button>
              <div v-if="subjectMenuOpen" class="ticket-form-dropdown" role="listbox">
                <button
                  v-for="item in askDynoItems"
                  :key="item.key"
                  type="button"
                  class="ticket-form-dropdown-item"
                  :class="{ active: item.label === ticketForm.subject }"
                  role="option"
                  :aria-selected="item.label === ticketForm.subject"
                  @click="selectSubject(item.label)"
                >
                  {{ item.label }}
                </button>
              </div>
            </div>
          </div>

          <div class="ticket-form-field">
            <label class="ticket-form-label">Bug Title</label>
            <input
              type="text"
              class="ticket-form-input"
              placeholder="Enter a short summary of the bug..."
              v-model="ticketForm.bugTitle"
            />
          </div>

          <div class="ticket-form-field">
            <label class="ticket-form-label">Description</label>
            <textarea
              class="ticket-form-input textarea"
              placeholder="Please describe what went wrong, what you were trying to do, and what happened instead.."
              v-model="ticketForm.description"
            ></textarea>
          </div>

          <div class="ticket-form-field">
            <label class="ticket-form-label">Expected Result</label>
            <textarea
              class="ticket-form-input textarea"
              placeholder="Please describe the expected result"
              v-model="ticketForm.expectedResult"
            ></textarea>
          </div>

          <div class="ticket-form-field">
            <label class="ticket-form-label">Priority</label>
            <div class="ticket-form-select" ref="priorityMenuRef">
              <button
                type="button"
                class="ticket-form-input select"
                :class="{ open: priorityMenuOpen, placeholder: !ticketForm.priority }"
                aria-haspopup="listbox"
                :aria-expanded="priorityMenuOpen"
                @click="togglePriorityMenu"
              >
                <span>{{ ticketForm.priority || 'Select priority' }}</span>
                <img class="icon-chevron-down" :src="iconChevronDown" alt="" />
              </button>
              <div v-if="priorityMenuOpen" class="ticket-form-dropdown" role="listbox">
                <button
                  v-for="option in priorityOptions"
                  :key="option"
                  type="button"
                  class="ticket-form-dropdown-item"
                  :class="{ active: option === ticketForm.priority }"
                  role="option"
                  :aria-selected="option === ticketForm.priority"
                  @click="selectPriority(option)"
                >
                  {{ option }}
                </button>
              </div>
            </div>
          </div>

          <div class="ticket-form-field">
            <label class="ticket-form-label">Phone Number</label>
            <input
              type="tel"
              class="ticket-form-input"
              placeholder="e.g. 012-23456789"
              v-model="ticketForm.phone"
            />
          </div>

          <div class="ticket-form-field">
            <label class="ticket-form-label">Upload file</label>
            <div class="ticket-form-upload" :class="{ 'has-files': selectedFiles.length > 0 }">
              <template v-if="selectedFiles.length === 0">
                <div class="ticket-form-upload-placeholder">
                  <img class="ticket-form-upload-icon" :src="iconUpload" alt="" />
                  <p class="ticket-form-upload-hint">
                    <span class="emphasis">Click to upload</span> or drag and drop
                  </p>
                  <p class="ticket-form-upload-caption">Max. File Size: 30MB &middot; Up to {{ MAX_UPLOAD_FILES }} images</p>
                </div>
                <button type="button" class="ticket-form-browse-button" @click="triggerFileBrowse">
                  <span>Browse file</span>
                </button>
              </template>
              <template v-else>
                <ul class="ticket-form-upload-list">
                  <li v-for="(file, index) in selectedFiles" :key="`${file.name}-${index}`" class="ticket-form-upload-list-item">
                    <span class="ticket-form-upload-list-name">{{ file.name }}</span>
                    <button
                      type="button"
                      class="ticket-form-upload-remove"
                      :aria-label="`Remove ${file.name}`"
                      @click="removeSelectedFile(index)"
                    >
                      <img class="icon-close" :src="iconClose" alt="" />
                    </button>
                  </li>
                </ul>
                <button
                  v-if="selectedFiles.length < MAX_UPLOAD_FILES"
                  type="button"
                  class="ticket-form-browse-button"
                  @click="triggerFileBrowse"
                >
                  <span>Add more ({{ selectedFiles.length }}/{{ MAX_UPLOAD_FILES }})</span>
                </button>
                <p v-else class="ticket-form-upload-caption">Maximum of {{ MAX_UPLOAD_FILES }} images reached</p>
              </template>
              <input
                ref="fileInputRef"
                type="file"
                accept="image/*"
                multiple
                class="ticket-form-file-input"
                @change="handleFileChange"
              />
            </div>
            <p class="ticket-form-upload-helper">SVG, PNG, JPG or GIF (MAX. 800x400px).</p>
          </div>
        </div>

        <div class="ticket-form-footer">
          <button type="button" class="ticket-form-button" @click="closeTicketForm">Cancel</button>
          <div class="ticket-form-footer-actions">
            <button type="button" class="ticket-form-button" @click="saveDraftTicket">Save Draft</button>
            <button type="button" class="ticket-form-button primary" @click="submitTicket">Submit Ticket</button>
          </div>
        </div>
      </div>

      <div v-else-if="ticketSubmitted" class="ticket-success-modal" role="dialog" aria-modal="true" aria-label="Ticket submitted" @click.stop>
        <button type="button" class="ticket-success-close" aria-label="Close" @click="closeTicketForm">
          <img class="icon-close" :src="iconClose" alt="" />
        </button>
        <div class="ticket-success-badge">
          <img class="ticket-success-pop-icon ticket-success-check" :src="iconCheckCircle" alt="" />
        </div>
        <div class="ticket-success-heading">
          <p class="ticket-success-title">Ticket Submitted</p>
          <p class="ticket-success-description">
            Your request has been successfully submitted to the MCDynect Support Team.
            <br />
            <br />
            Our team will review your request and update you through system chat.
          </p>
        </div>
        <p class="ticket-success-number">Ticket No. : {{ submittedTicketNumber }}</p>
        <div class="ticket-success-footer">
          <button type="button" class="ticket-success-button" @click="closeTicketForm">Done</button>
          <button type="button" class="ticket-success-button primary" @click="viewSubmittedTicket">View Ticket</button>
        </div>
      </div>

      <div v-else class="ticket-success-modal" role="dialog" aria-modal="true" aria-label="Draft saved" @click.stop>
        <button type="button" class="ticket-success-close" aria-label="Close" @click="closeTicketForm">
          <img class="icon-close" :src="iconClose" alt="" />
        </button>
        <div class="ticket-success-badge warning">
          <img class="ticket-success-pop-icon ticket-success-warning" :src="iconWarningTriangle" alt="" />
        </div>
        <div class="ticket-success-heading">
          <p class="ticket-success-title">Draft Saved</p>
          <p class="ticket-success-description">
            Your ticket has been saved as a draft. It won't be sent to the support team until you submit it.
          </p>
        </div>
        <p class="ticket-success-number">Draft ID: {{ savedDraftId }}</p>
        <div class="ticket-success-footer">
          <button type="button" class="ticket-success-button" @click="closeTicketForm">Done</button>
          <button type="button" class="ticket-success-button primary" @click="editDraft">Edit Draft</button>
        </div>
      </div>
    </div>
  </Teleport>
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
.icon-button:hover .icon-sidebar-toggle,
.icon-button:hover .icon-bell,
.icon-button:hover .icon-close {
  /* Recolors the #4A5565 source icon to the primary #b13444, calibrated to an exact match. */
  filter: invert(18%) sepia(95%) saturate(137%) hue-rotate(303deg) brightness(92%) contrast(273%);
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
.icon-close {
  width: 11.67px;
  height: 11.67px;
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
  color: #b13444;
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
.notification-dismiss:hover .icon-trash {
  filter: invert(18%) sepia(95%) saturate(137%) hue-rotate(303deg) brightness(92%) contrast(273%);
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
  color: #b13444;
}
.account-dropdown-item:hover .icon-user,
.account-dropdown-item:hover .icon-question-circle {
  filter: invert(18%) sepia(95%) saturate(137%) hue-rotate(303deg) brightness(92%) contrast(273%);
}
.account-dropdown-item.danger {
  color: #c70036;
}
.account-dropdown-item.danger:hover {
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
.ask-dyno-button.open {
  background-size: 100% 100%;
  border-color: #b13444;
  color: #ffffff;
}

.ask-dyno-menu {
  position: relative;
}

.ask-dyno-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  gap: 6px;
  width: 320px;
  padding: 8px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 10px 15px -3px rgba(29, 41, 61, 0.1), 0px 4px 6px -4px rgba(29, 41, 61, 0.1);
}

.ask-dyno-cta {
  position: relative;
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 12px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
}
/* Graph-paper texture behind the CTA copy — reproduced as two hairline
   gradients instead of Figma's exported version (thousands of individual
   1px-gap square paths, ~400KB for a decorative background). */
.ask-dyno-cta-pattern {
  position: absolute;
  inset: 0;
  background-image: linear-gradient(to right, #eef0f3 1px, transparent 1px),
    linear-gradient(to bottom, #eef0f3 1px, transparent 1px);
  background-size: 7.22px 7.55px;
}

.ask-dyno-cta-label {
  position: relative;
  display: flex;
  flex: 1 0 0;
  flex-direction: column;
  gap: 6px;
  align-items: flex-start;
  min-width: 0;
}

.ask-dyno-cta-heading {
  font-size: 14px;
  font-weight: 500;
  line-height: 24px;
  color: #101828;
}

.view-ticket-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 6px 12px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 12px;
  font-weight: 500;
  line-height: 20px;
  color: #4a5565;
  cursor: pointer;
  transition: background 0.15s ease;
}
.view-ticket-button:hover {
  background: #b13444;
  color: #ffffff;
}
.view-ticket-button:hover .icon-arrow-right {
  filter: brightness(0) invert(1);
}
.view-ticket-button .icon-arrow-right {
  width: 9.33px;
  height: 5.83px;
}

.ask-dyno-cta-count {
  position: relative;
  display: flex;
  flex-shrink: 0;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1px;
  padding: 12px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  text-align: center;
}
.ask-dyno-cta-count-number {
  font-family: inherit;
  font-weight: 700;
  font-size: 16px;
  line-height: 1;
  letter-spacing: -0.4px;
  color: #101828;
}
.ask-dyno-cta-count-label {
  font-weight: 400;
  font-size: 12px;
  line-height: 1.5;
  color: #4a5565;
}

.ask-dyno-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  width: 100%;
}

.ask-dyno-item {
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
  line-height: 1;
  color: #4a5565;
  text-align: left;
  white-space: nowrap;
  cursor: pointer;
  transition: background 0.15s ease;
}
.ask-dyno-item:hover {
  background: rgba(249, 223, 227, 0.5);
  color: #b13444;
}
/* 16x16 alignment slot, same convention as .account-dropdown-icon: each
   glyph keeps its own exported (non-square) aspect ratio inside it. */
.ask-dyno-item-icon {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: 16px;
  height: 16px;
}
/* Inline SVG with fill="currentColor" (same convention as .icon-headset
   above) so the glyph recolors with the item's text on hover, instead of
   staying pinned to a color baked into an exported <img> asset. */
.ask-dyno-item-icon svg {
  display: block;
  flex-shrink: 0;
}
.ask-dyno-item-icon .icon-bug {
  width: 13.33px;
  height: 11.33px;
}
.ask-dyno-item-icon .icon-clipboard {
  width: 10.67px;
  height: 13.33px;
}
.ask-dyno-item-icon .icon-lightbulb {
  width: 9.33px;
  height: 13.33px;
}
.ask-dyno-item-icon .icon-life-saver {
  width: 13.34px;
  height: 13.34px;
}
.ask-dyno-item-icon .icon-lock-open {
  width: 12px;
  height: 12px;
}
.ask-dyno-item-icon .icon-dots-vertical {
  width: 2px;
  height: 10px;
}

.ask-dyno-separator {
  display: block;
  height: 1px;
  background: #f3f4f6;
}

/* Ticket Form modal — opened from any Ask Dyno menu item
   (Figma node 1466-9310). Teleported to <body> so it isn't clipped by the
   topnav's own fixed positioning/stacking context. */
.ticket-form-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  background: rgba(16, 24, 40, 0.5);
}

.ticket-form-modal {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 673px;
  max-width: 100%;
  max-height: 90vh;
  padding: 24px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
}

.ticket-form-header {
  display: flex;
  gap: 24px;
  align-items: flex-start;
  flex-shrink: 0;
  width: 100%;
  padding-bottom: 20px;
  border-bottom: 1px solid #e5e7eb;
}

.ticket-form-heading {
  display: flex;
  flex: 1 0 0;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
}

.ticket-form-title {
  font-size: 16px;
  font-weight: 600;
  line-height: 28px;
  color: #101828;
}

.ticket-form-subtitle {
  font-size: 14px;
  font-weight: 400;
  color: #4a5565;
}

.ticket-form-body {
  display: flex;
  flex: 1 1 auto;
  flex-direction: column;
  gap: 16px;
  min-height: 0;
  overflow-y: auto;
}

.ticket-form-field {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
}

.ticket-form-label {
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: #101828;
}
.ticket-form-label .required {
  color: #c70036;
}

.ticket-form-input {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
  padding: 10px 12px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  color: #101828;
}
.ticket-form-input::placeholder {
  color: #6a7282;
}
.ticket-form-input.select {
  justify-content: space-between;
  color: #101828;
  text-align: left;
  cursor: pointer;
}
.ticket-form-input.select.placeholder span:first-child {
  color: #6a7282;
}
textarea.ticket-form-input.textarea {
  min-height: 64px;
  resize: vertical;
  line-height: 20px;
}

.ticket-form-select {
  position: relative;
  width: 100%;
}

.icon-chevron-down {
  width: 4px;
  height: 6.67px;
  flex-shrink: 0;
  transform: rotate(-90deg);
}

.ticket-form-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  right: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  gap: 6px;
  max-height: 220px;
  padding: 8px;
  overflow-y: auto;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 10px 15px -3px rgba(29, 41, 61, 0.1), 0px 4px 6px -4px rgba(29, 41, 61, 0.1);
}

.ticket-form-dropdown-item {
  padding: 8px;
  border: none;
  border-radius: 8px;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: #4a5565;
  text-align: left;
  cursor: pointer;
  transition: background 0.15s ease;
}
.ticket-form-dropdown-item:hover {
  background: rgba(249, 223, 227, 0.5);
  color: #b13444;
}
.ticket-form-dropdown-item.active {
  background: #fae8ea;
  color: #b13444;
}

.ticket-form-upload {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  width: 100%;
  min-height: 228px;
  padding: 20px 16px;
  background: #f9fafb;
  border: 1px dashed #e5e7eb;
  border-radius: 12px;
}
.ticket-form-upload.has-files {
  justify-content: flex-start;
}

.ticket-form-upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.ticket-form-upload-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
  max-height: 160px;
  overflow-y: auto;
}

.ticket-form-upload-list-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  width: 100%;
  padding: 8px 10px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
}

.ticket-form-upload-list-name {
  flex: 1 0 0;
  min-width: 0;
  overflow: hidden;
  font-size: 13px;
  color: #4a5565;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.ticket-form-upload-remove {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  padding: 0;
  border: none;
  border-radius: 8px;
  background: transparent;
  cursor: pointer;
  transition: background 0.15s ease;
}
.ticket-form-upload-remove:hover {
  background: rgba(249, 223, 227, 0.5);
}
.ticket-form-upload-remove:hover .icon-close {
  filter: invert(18%) sepia(95%) saturate(137%) hue-rotate(303deg) brightness(92%) contrast(273%);
}
.ticket-form-upload-remove .icon-close {
  width: 9px;
  height: 9px;
}

.ticket-form-upload-icon {
  width: 24px;
  height: 24px;
}

.ticket-form-upload-hint {
  font-size: 14px;
  line-height: 20px;
  color: #6a7282;
  text-align: center;
}
.ticket-form-upload-hint .emphasis {
  font-weight: 500;
}

.ticket-form-upload-caption {
  font-size: 12px;
  font-weight: 500;
  line-height: 16px;
  color: #6a7282;
  text-align: center;
}

.ticket-form-browse-button {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: #911b2d;
  border: none;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: #ffffff;
  cursor: pointer;
}
.ticket-form-browse-button:hover {
  background: #b13444;
}

.ticket-form-upload-helper {
  font-size: 12px;
  line-height: 20px;
  color: #4a5565;
}

.ticket-form-file-input {
  display: none;
}

.ticket-form-footer {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding-top: 24px;
  border-top: 1px solid #e5e7eb;
}

.ticket-form-footer-actions {
  display: flex;
  gap: 16px;
  align-items: center;
}

.ticket-form-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 10px 16px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: #4a5565;
  cursor: pointer;
  transition: background 0.15s ease;
}
.ticket-form-button:hover {
  background: rgba(249, 223, 227, 0.5);
  color: #b13444;
}
.ticket-form-button.primary {
  background: #911b2d;
  border-color: #911b2d;
  color: #ffffff;
}
.ticket-form-button.primary:hover {
  background: #b13444;
  border-color: #b13444;
  color: #ffffff;
}

/* Ticket Submitted success panel (Figma node 1472-9672), shown in place of
   the form once submitTicket() resolves. */
.ticket-success-modal {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  width: 400px;
  max-width: 100%;
  padding: 24px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
}

.ticket-success-close {
  position: absolute;
  top: 5px;
  right: 5px;
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
.ticket-success-close:hover {
  background: rgba(249, 223, 227, 0.5);
}
.ticket-success-close:hover .icon-close {
  filter: invert(18%) sepia(95%) saturate(137%) hue-rotate(303deg) brightness(92%) contrast(273%);
}

.ticket-success-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 48px;
  height: 48px;
  background: #ecfdf5;
  border-radius: 9999px;
}
.ticket-success-badge.warning {
  background: #fefce8;
}

/* Figma's badge icon pops in with a damped-spring scale (sampled from the
   custom easing motion.dev returned) and loops every 2s per its
   `repeat: Infinity` transition. Shared by the checkmark and warning icons. */
.ticket-success-pop-icon {
  animation:
    ticket-success-fade 2s infinite,
    ticket-success-scale 2s infinite;
  transform-origin: center;
}
@media (prefers-reduced-motion: reduce) {
  .ticket-success-pop-icon {
    animation: none;
  }
}

.ticket-success-check {
  width: 18.67px;
  height: 12.83px;
}

.ticket-success-warning {
  width: 23.33px;
  height: 23.33px;
}

@keyframes ticket-success-fade {
  0% {
    opacity: 0;
    animation-timing-function: ease-out;
  }
  6% {
    opacity: 1;
  }
  100% {
    opacity: 1;
  }
}

@keyframes ticket-success-scale {
  0% {
    transform: scale(0);
    animation-timing-function: linear(0, 0.444, 0.939, 1.093, 1.062, 1.012, 0.992, 0.993, 0.998, 1.001, 1.001);
  }
  25% {
    transform: scale(1);
  }
  100% {
    transform: scale(1);
  }
}

.ticket-success-heading {
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: center;
  text-align: center;
}

.ticket-success-title {
  font-size: 16px;
  font-weight: 600;
  line-height: 24px;
  color: #101828;
}

.ticket-success-description {
  font-size: 16px;
  font-weight: 400;
  line-height: 24px;
  color: #4a5565;
}

.ticket-success-number {
  width: 100%;
  font-size: 16px;
  font-weight: 600;
  line-height: 24px;
  color: #af2136;
  text-align: center;
}

.ticket-success-footer {
  display: flex;
  gap: 16px;
  align-items: center;
  justify-content: center;
  width: 100%;
}

.ticket-success-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 8px 16px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: #4a5565;
  cursor: pointer;
  transition: background 0.15s ease;
}
.ticket-success-button:hover {
  background: rgba(249, 223, 227, 0.5);
  color: #b13444;
}
.ticket-success-button.primary {
  background: #911b2d;
  border-color: #911b2d;
  color: #ffffff;
}
.ticket-success-button.primary:hover {
  background: #b13444;
  border-color: #b13444;
  color: #ffffff;
}
</style>
