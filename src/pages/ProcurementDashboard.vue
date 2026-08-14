<script setup>
/**
 * ProcurementDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1115-25558
 *
 * Procurement module landing dashboard: sidebar + topnav shell around order-status,
 * consumption, pickup-method, and preorder-trend widgets. Reached from the workspace
 * selector's "Procurement" module card. The topnav's workspace switcher ("change role")
 * is hidden here, same as Finance/Licensing.
 */
import { onBeforeUnmount, onMounted, ref } from 'vue'
import VueApexCharts from 'vue3-apexcharts'
import DashboardSidebar from '../components/DashboardSidebar.vue'
import DashboardTopnav from '../components/DashboardTopnav.vue'

import iconAngleDown from '../assets/dashboard/icons/icon-angle-down.svg'
import iconCalendarMonth from '../assets/dashboard/icons/icon-calendar-month.svg'
import iconLayers from '../assets/dashboard/icons/icon-layers.svg'
import iconMinus from '../assets/dashboard/icons/icon-minus.svg'
import iconChevronDetails from '../assets/dashboard/icons/icon-chevron-see-details.svg'
import iconStatTotalExpenses from '../assets/dashboard/icons/icon-stat-total-expenses.svg'
import iconStatOtherExpenses from '../assets/dashboard/icons/icon-stat-other-expenses.svg'
import iconStatNetProfit from '../assets/dashboard/icons/icon-stat-net-profit.svg'

import iconGrid from '../assets/dashboard/sidebar/licensing/icon-grid.svg'
import iconTag from '../assets/dashboard/sidebar/licensing/icon-tag.svg'
import iconCart from '../assets/dashboard/sidebar/licensing/icon-cart.svg'
import iconFileLines from '../assets/dashboard/sidebar/licensing/icon-file-lines.svg'
import iconClipboardList from '../assets/dashboard/sidebar/licensing/icon-clipboard-list.svg'
import iconBuilding from '../assets/dashboard/sidebar/finance/icon-building.svg'
import iconSettings from '../assets/dashboard/sidebar/icon-settings.svg'
import iconFolder from '../assets/dashboard/sidebar/procurement/icon-folder.svg'
import iconFolderDuplicate from '../assets/dashboard/sidebar/procurement/icon-folder-duplicate.svg'
import iconArchive from '../assets/dashboard/sidebar/procurement/icon-archive.svg'
import iconClipboardCheck from '../assets/dashboard/sidebar/procurement/icon-clipboard-check.svg'

import imgPatternBg from '../assets/dashboard/pattern-bg.jpg'

const emit = defineEmits(['back-to-workspace', 'sign-out'])

// Sidebar module list for the Procurement workspace. None of these open
// sub-navigation in the Figma design (no chevron affordance).
const navItems = [
  { key: 'overview', icon: iconGrid, label: 'Overview', width: 15, height: 15 },
  { key: 'manage-items', icon: iconTag, label: 'Manage Items', width: 15, height: 15 },
  { key: 'order-request', icon: iconCart, label: 'Order Request', width: 13.33, height: 15 },
  { key: 'purchase-quotations', icon: iconFileLines, label: 'Purchase Quotations', width: 13.33, height: 16.67 },
  { key: 'purchase-requisitions', icon: iconClipboardList, label: 'Purchase Requisitions', width: 13.33, height: 16.67 },
  { key: 'manage-customized-product', icon: iconSettings, label: 'Manage Customized Product', width: 16.67, height: 16.67 },
  { key: 'product-categories', icon: iconFolder, label: 'Product Categories', width: 15, height: 14.17 },
  { key: 'item-category', icon: iconFolderDuplicate, label: 'Item Category', width: 15, height: 15 },
  { key: 'vendor-list', icon: iconBuilding, label: 'Vendor List', width: 13.33, height: 15 },
  { key: 'purchase-order-deliveries', icon: iconArchive, label: 'Purchase Order Deliveries', width: 16.67, height: 13.33 },
  { key: 'vendor-review', icon: iconClipboardCheck, label: 'Vendor Review', width: 13.33, height: 16.67 },
]

const activeNavItem = ref('overview')
const sidebarCollapsed = ref(false)

function handleNavigate(key) {
  // Only Overview is built; the rest are no-ops for now.
  if (key === 'overview') activeNavItem.value = key
}

const filterMenuOpen = ref(false)
const filterMenuRef = ref(null)

const outletOptions = ['All', 'Mr Churros Bandar Sri Uda', 'Mr Churros Taman Melati', 'Mr Churros Kota Damansara']
const stateOptions = ['All', 'Selangor', 'Johor', 'Penang', 'Kedah']

const filterOutlet = ref('All')
const filterState = ref('All')
const filterCalendar = ref('')

function toggleFilterMenu() {
  filterMenuOpen.value = !filterMenuOpen.value
}

function resetFilters() {
  filterOutlet.value = 'All'
  filterState.value = 'All'
  filterCalendar.value = ''
}

function applyFilters() {
  // Filtering logic isn't wired to the widgets yet — closing the menu is the only effect for now.
  filterMenuOpen.value = false
}

const layersPanelOpen = ref(false)
const layersButtonRef = ref(null)
const layersPanelRef = ref(null)

const pendingActivity = [
  { index: '1', category: 'Purchase Requisition Approvals', count: '(8 Items)' },
  { index: '2', category: 'Vendor Registration Review', count: '(5 Items)' },
  { index: '3', category: 'Purchase Order Deliveries', count: '(11 Items)' },
  { index: '4', category: 'Purchase Quotation Follow-ups', count: '(4 Items)' },
  { index: '5', category: 'Customized Product Requests', count: '(3 Items)' },
]

function toggleLayersPanel() {
  layersPanelOpen.value = !layersPanelOpen.value
}

function handleClickOutside(event) {
  if (filterMenuRef.value && !filterMenuRef.value.contains(event.target)) {
    filterMenuOpen.value = false
  }
  if (
    layersButtonRef.value &&
    !layersButtonRef.value.contains(event.target) &&
    layersPanelRef.value &&
    !layersPanelRef.value.contains(event.target)
  ) {
    layersPanelOpen.value = false
  }
}

onMounted(() => document.addEventListener('click', handleClickOutside))
onBeforeUnmount(() => document.removeEventListener('click', handleClickOutside))

const statCards = [
  { key: 'total-expenses', label: 'Total Expenses', value: 'MYR 2,845,120', delta: '0%', bg: '#fef2f2', icon: iconStatTotalExpenses, iconW: 15, iconH: 15 },
  { key: 'other-expenses', label: 'Other Expenses', value: 'MYR 21,650', delta: '0%', bg: '#fffbeb', icon: iconStatOtherExpenses, iconW: 11.67, iconH: 16.67 },
  { key: 'net-profit', label: 'Net Profit', value: '9,340', delta: '0%', bg: '#f0fdf4', icon: iconStatNetProfit, iconW: 15, iconH: 14.58 },
]

function formatMyr(value) {
  return `MYR ${Math.round(value).toLocaleString('en-MY')}`
}

// Other Status Funnel — concentric radial-bar rings for order status breakdown.
// ApexCharts' radialBar type doesn't support hover tooltips on the ring
// segments the way bar/donut/area do (no listeners are attached to the arc
// paths), so hover is tracked manually below and rendered with the same
// .chart-tooltip markup the other widgets use.
const orderStatusBreakdown = [
  { label: 'Pending', value: 92, color: '#fdc700' },
  { label: 'Processing', value: 68, color: '#1447e6' },
  { label: 'Done', value: 45, color: '#05df72' },
]

const orderStatusChartOptions = {
  chart: {
    type: 'radialBar',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  plotOptions: {
    radialBar: {
      hollow: { size: '35%' },
      track: { background: '#f3f4f6', margin: 6 },
      dataLabels: { show: false },
    },
  },
  stroke: { lineCap: 'round' },
  colors: orderStatusBreakdown.map((status) => status.color),
  labels: orderStatusBreakdown.map((status) => status.label),
  tooltip: { enabled: false },
}

const orderStatusSeries = orderStatusBreakdown.map((status) => status.value)
const orderStatusHollowRatio = 0.35

const radialChartRef = ref(null)
const radialHover = ref(null)

function handleRadialMouseMove(event) {
  const svg = radialChartRef.value?.querySelector('svg')
  if (!svg) return

  const svgRect = svg.getBoundingClientRect()
  const containerRect = radialChartRef.value.getBoundingClientRect()
  const outerRadius = Math.min(svgRect.width, svgRect.height) / 2
  const hollowRadius = outerRadius * orderStatusHollowRatio

  const dx = event.clientX - (svgRect.left + svgRect.width / 2)
  const dy = event.clientY - (svgRect.top + svgRect.height / 2)
  const distance = Math.sqrt(dx * dx + dy * dy)

  if (distance < hollowRadius || distance > outerRadius) {
    radialHover.value = null
    return
  }

  // Ring 0 (Pending) is outermost, so band index runs opposite to distance.
  const ringBand = (outerRadius - hollowRadius) / orderStatusBreakdown.length
  const ringIndex = Math.min(
    orderStatusBreakdown.length - 1,
    Math.floor((outerRadius - distance) / ringBand)
  )
  const status = orderStatusBreakdown[ringIndex]

  radialHover.value = {
    label: status.label,
    value: status.value,
    x: event.clientX - containerRect.left,
    y: event.clientY - containerRect.top,
  }
}

function handleRadialMouseLeave() {
  radialHover.value = null
}

// Consumption Analysis — daily bar chart, bars alternate between two blues.
const consumptionByDay = [
  { day: 'Mon', value: 113 },
  { day: 'Tue', value: 138 },
  { day: 'Wed', value: 152 },
  { day: 'Thu', value: 130 },
  { day: 'Fri', value: 105 },
  { day: 'Sat', value: 96 },
  { day: 'Sun', value: 139 },
]

const consumptionChartOptions = {
  chart: {
    type: 'bar',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  plotOptions: {
    bar: {
      columnWidth: '55%',
      borderRadius: 4,
      distributed: true,
    },
  },
  colors: consumptionByDay.map((_, i) => (i % 2 === 0 ? '#1447e6' : '#8ec5ff')),
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: consumptionByDay.map((point) => point.day),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[0][dataPointIndex]} units</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const consumptionSeries = [{ name: 'Consumption', data: consumptionByDay.map((point) => point.value) }]

// Pickup Method Split — donut chart with a center readout.
const pickupMethodBreakdown = [
  { label: 'Self-pickup', value: 82.4, color: '#ff8904' },
  { label: 'Transport', value: 17.6, color: '#51a2ff' },
]

const pickupMethodChartOptions = {
  chart: {
    type: 'donut',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  labels: pickupMethodBreakdown.map((method) => method.label),
  colors: pickupMethodBreakdown.map((method) => method.color),
  dataLabels: { enabled: false },
  legend: { show: false },
  stroke: { show: false },
  plotOptions: {
    pie: {
      donut: {
        size: '75%',
        labels: { show: false },
      },
    },
  },
  tooltip: {
    custom: ({ series, seriesIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex]}%</div>
        <div class="chart-tooltip-label">${w.globals.labels[seriesIndex]}</div>
      </div>
    `,
  },
}

const pickupMethodSeries = pickupMethodBreakdown.map((method) => method.value)

// Preorder Volume Trend — smoothed area chart across the week.
const preorderVolumeByDay = [
  { day: 'Mon', value: 480 },
  { day: 'Tue', value: 620 },
  { day: 'Wed', value: 540 },
  { day: 'Thu', value: 380 },
  { day: 'Fri', value: 590 },
  { day: 'Sat', value: 780 },
  { day: 'Sun', value: 520 },
]

const preorderVolumeChartOptions = {
  chart: {
    type: 'area',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  colors: ['#1447e6'],
  stroke: { curve: 'smooth', width: 2 },
  fill: {
    type: 'gradient',
    gradient: { shadeIntensity: 1, opacityFrom: 0.35, opacityTo: 0, stops: [0, 90, 100] },
  },
  markers: { size: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: preorderVolumeByDay.map((point) => point.day),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]} preorders</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const preorderVolumeSeries = [{ name: 'Preorders', data: preorderVolumeByDay.map((point) => point.value) }]
</script>

<template>
  <div class="dashboard" :class="{ 'sidebar-collapsed': sidebarCollapsed, 'layers-panel-open': layersPanelOpen }">
    <DashboardSidebar
      :active-item="activeNavItem"
      :collapsed="sidebarCollapsed"
      :nav-items="navItems"
      @navigate="handleNavigate"
    />

    <div class="dashboard-main">
      <div class="dashboard-bg" :style="{ backgroundImage: 'url(' + imgPatternBg + ')' }"></div>

      <DashboardTopnav
        :collapsed="sidebarCollapsed"
        :show-workspace-switcher="false"
        @toggle-sidebar="sidebarCollapsed = !sidebarCollapsed"
        @sign-out="emit('sign-out')"
      />

      <div class="dashboard-content">
        <div class="header-section">
          <div class="heading-block">
            <h1 class="heading">Hello, Izzthdr.</h1>
            <p class="subheading">Here's what's happening today.</p>
          </div>
          <div class="header-actions">
            <div class="filter-menu" ref="filterMenuRef">
              <button
                type="button"
                class="filter-button"
                :class="{ open: filterMenuOpen }"
                aria-haspopup="true"
                :aria-expanded="filterMenuOpen"
                @click="toggleFilterMenu"
              >
                <span>Filter</span>
                <img :src="iconAngleDown" alt="" />
              </button>
              <div v-if="filterMenuOpen" class="filter-dropdown" role="menu">
                <div class="filter-field">
                  <label class="filter-label">Outlet</label>
                  <div class="filter-select">
                    <select v-model="filterOutlet">
                      <option v-for="option in outletOptions" :key="option" :value="option">{{ option }}</option>
                    </select>
                    <img class="filter-select-icon" :src="iconAngleDown" alt="" />
                  </div>
                </div>
                <div class="filter-field">
                  <label class="filter-label">State</label>
                  <div class="filter-select">
                    <select v-model="filterState">
                      <option v-for="option in stateOptions" :key="option" :value="option">{{ option }}</option>
                    </select>
                    <img class="filter-select-icon" :src="iconAngleDown" alt="" />
                  </div>
                </div>
                <div class="filter-field">
                  <label class="filter-label">Calendar</label>
                  <div class="filter-select">
                    <input
                      v-model="filterCalendar"
                      type="text"
                      class="filter-date-input"
                      placeholder="Select your calendar"
                    />
                    <img class="filter-select-icon filter-calendar-icon" :src="iconCalendarMonth" alt="" />
                  </div>
                </div>
                <div class="filter-actions">
                  <button type="button" class="filter-reset-button" @click="resetFilters">Reset</button>
                  <button type="button" class="filter-apply-button" @click="applyFilters">Apply Filter</button>
                </div>
              </div>
            </div>
            <button
              ref="layersButtonRef"
              type="button"
              class="layers-button"
              aria-label="View layers"
              aria-haspopup="true"
              :aria-expanded="layersPanelOpen"
              @click="toggleLayersPanel"
            >
              <img :src="iconLayers" alt="" />
            </button>
          </div>
        </div>

        <div class="stat-row">
          <div v-for="stat in statCards" :key="stat.key" class="stat-card">
            <div class="stat-data">
              <p class="stat-label">{{ stat.label }}</p>
              <p class="stat-value">{{ stat.value }}</p>
              <div class="stat-delta">
                <img class="delta-icon" :src="iconMinus" alt="" />
                <span class="delta-value">{{ stat.delta }}</span>
                <span class="delta-caption">vs last month</span>
              </div>
            </div>
            <div class="stat-icon-wrap" :style="{ background: stat.bg }">
              <img :src="stat.icon" :style="{ width: stat.iconW + 'px', height: stat.iconH + 'px' }" alt="" />
            </div>
          </div>
        </div>

        <div class="widget-grid">
          <div class="widget-card">
            <div class="card-header">
              <h2>Other Status Funnel</h2>
              <p>Breakdown of order status across the pipeline.</p>
            </div>
            <div
              class="radial-chart"
              ref="radialChartRef"
              @mousemove="handleRadialMouseMove"
              @mouseleave="handleRadialMouseLeave"
            >
              <VueApexCharts
                type="radialBar"
                width="100%"
                height="100%"
                :options="orderStatusChartOptions"
                :series="orderStatusSeries"
              />
              <div
                v-if="radialHover"
                class="chart-tooltip radial-tooltip"
                :style="{ left: radialHover.x + 'px', top: radialHover.y + 'px' }"
              >
                <div class="chart-tooltip-value">{{ radialHover.value }}%</div>
                <div class="chart-tooltip-label">{{ radialHover.label }}</div>
              </div>
            </div>
            <div class="widget-legend">
              <div v-for="status in orderStatusBreakdown" :key="status.label" class="legend-item">
                <span class="legend-dot" :style="{ background: status.color }"></span>
                <span>{{ status.label }}</span>
              </div>
            </div>
            <div class="card-footer">
              <span>Last Updated : Today, 2:45PM</span>
              <a href="#" class="see-details">
                See Details
                <img :src="iconChevronDetails" alt="" />
              </a>
            </div>
          </div>

          <div class="widget-card">
            <div class="card-header">
              <h2>Consumption Analysis</h2>
              <p>Daily consumption levels across the week.</p>
            </div>
            <div class="bar-chart">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="consumptionChartOptions"
                :series="consumptionSeries"
              />
            </div>
            <div class="card-footer">
              <span>Last Updated : Today, 2:45PM</span>
              <a href="#" class="see-details">
                See Details
                <img :src="iconChevronDetails" alt="" />
              </a>
            </div>
          </div>

          <div class="widget-card donut-card">
            <div class="card-header">
              <h2>Pickup Method Split</h2>
              <p>Distribution of orders by pickup method.</p>
            </div>
            <div class="donut-chart">
              <VueApexCharts
                type="donut"
                width="100%"
                height="100%"
                :options="pickupMethodChartOptions"
                :series="pickupMethodSeries"
              />
              <div class="donut-label">
                <p class="donut-percent">{{ pickupMethodBreakdown[0].value }}%</p>
                <p class="donut-caption">Self Pickup</p>
              </div>
            </div>
            <div class="widget-legend">
              <div v-for="method in pickupMethodBreakdown" :key="method.label" class="legend-item">
                <span class="legend-dot" :style="{ background: method.color }"></span>
                <span>{{ method.label }}</span>
              </div>
            </div>
            <div class="card-footer">
              <span>Last Updated : Today, 2:45PM</span>
              <a href="#" class="see-details">
                See Details
                <img :src="iconChevronDetails" alt="" />
              </a>
            </div>
          </div>

          <div class="widget-card">
            <div class="card-header">
              <h2>Preorder Volume Trend</h2>
              <p>Preorder volume trends across the week.</p>
            </div>
            <div class="bar-chart">
              <VueApexCharts
                type="area"
                width="100%"
                height="100%"
                :options="preorderVolumeChartOptions"
                :series="preorderVolumeSeries"
              />
            </div>
            <div class="card-footer">
              <span>Last Updated : Today, 2:45PM</span>
              <a href="#" class="see-details">
                See Details
                <img :src="iconChevronDetails" alt="" />
              </a>
            </div>
          </div>
        </div>
      </div>
    </div>

    <aside v-if="layersPanelOpen" ref="layersPanelRef" class="layers-panel">
      <p class="layers-panel-title">Pending Activity</p>
      <ol class="layers-panel-list">
        <li v-for="(item, i) in pendingActivity" :key="i" class="layers-panel-row">
          <span class="layers-panel-category">{{ item.index }}. {{ item.category }}</span>
          <span class="layers-panel-count">{{ item.count }}</span>
        </li>
      </ol>
    </aside>
  </div>
</template>

<style scoped>
.dashboard {
  --brand: #b13444;
  --heading: #101828;
  --body-text: #4a5565;
  --border: #e5e7eb;
  --border-light: #f3f4f6;
  --success: #007a55;
  --body-text-subtle: #6a7282;

  display: flex;
  align-items: stretch;
  width: 100%;
  min-height: 100vh;
  background: #F9FAFB;
  font-family: inherit;
}
.dashboard * {
  box-sizing: border-box;
}
.dashboard.sidebar-collapsed .dashboard-main {
  margin-left: 64px;
}
.dashboard.layers-panel-open .dashboard-main {
  margin-right: 267px;
}

.dashboard-main {
  position: relative;
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
  overflow: hidden;
  margin-left: 251px;
  margin-right: 0;
  transition: margin-left 220ms ease, margin-right 220ms ease;
}

.dashboard-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 500px;
  z-index: 0;
  background-size: cover;
  background-position: top;
  background-repeat: no-repeat;
  pointer-events: none;
}
.dashboard-bg::after {
  content: '';
  position: absolute;
  inset: 0;
  background-image: linear-gradient(
    180deg,
    rgba(249, 250, 251, 0) 0%,
    rgba(249, 250, 251, 0) 0%,
    rgba(249, 250, 251, 0.6) 60%,
    rgba(249, 250, 251, 0.946) 95%
    rgb(249, 250, 251) 100%
  );
}

.dashboard-content {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  gap: 24px;
  margin-top: 66px;
  padding: 32px;
}

/* Header */
.header-section {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}
.heading {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  line-height: 30px;
  color: var(--heading);
}
.subheading {
  margin: 6px 0 0;
  font-size: 14px;
  color: var(--body-text);
}
.header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-shrink: 0;
}
.filter-menu {
  position: relative;
}
.filter-button {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 16px;
  background: #ffffff;
  border: 1px solid #f4f2ef;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
  cursor: pointer;
}
.filter-button img {
  width: 11px;
  height: auto;
  transform: scaleY(-1);
  transition: transform 0.15s ease;
}
.filter-button.open img {
  transform: scaleY(1);
}

.filter-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  z-index: 20;
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 296px;
  padding: 16px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: 0px 10px 15px -3px rgba(29, 41, 61, 0.1), 0px 4px 6px -4px rgba(29, 41, 61, 0.1);
}
.filter-field {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
}
.filter-label {
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
}
.filter-select {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
  padding: 10px 12px;
  background: #f9fafb;
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
}
.filter-select select,
.filter-date-input {
  flex: 1;
  width: 100%;
  padding: 0;
  border: none;
  background: transparent;
  font-family: inherit;
  font-size: 14px;
  color: var(--body-text-subtle);
  cursor: pointer;
  appearance: none;
}
.filter-date-input {
  cursor: text;
}
.filter-date-input::placeholder {
  color: var(--body-text-subtle);
  opacity: 1;
}
.filter-select-icon {
  flex-shrink: 0;
  pointer-events: none;
}
.filter-select-icon:not(.filter-calendar-icon) {
  width: 10.67px;
  height: 6px;
}
.filter-calendar-icon {
  width: 12px;
  height: 12px;
}
.filter-select select ~ .filter-select-icon {
  transform: scaleY(-1);
}

.filter-actions {
  display: flex;
  gap: 12px;
  width: 100%;
}
.filter-reset-button,
.filter-apply-button {
  flex: 1;
  padding: 8px 12px;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
}
.filter-reset-button {
  background: #f9fafb;
  border: 1px solid var(--border);
  color: var(--body-text);
}
.filter-apply-button {
  background: var(--brand);
  border: none;
  color: #ffffff;
}
.layers-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  padding: 0;
  background: var(--brand);
  border: none;
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
  cursor: pointer;
}
.layers-button img {
  width: 20px;
  height: 20px;
}

/* Pending activity panel (opened via the header's layers button) */
.layers-panel {
  position: fixed;
  top: 66px;
  right: 0;
  bottom: 0;
  z-index: 15;
  display: flex;
  flex-direction: column;
  gap: 32px;
  width: 267px;
  padding: 32px 16px 28px;
  background: #ffffff;
  border-left: 1px solid var(--border);
  overflow-y: auto;
}
.layers-panel-title {
  margin: 0;
  font-size: 14px;
  font-weight: 600;
  line-height: 20px;
  color: var(--body-text);
}
.layers-panel-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin: 0;
  padding: 0;
  list-style: none;
}
.layers-panel-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}
.layers-panel-category {
  font-size: 14px;
  line-height: 20px;
  color: var(--heading);
}
.layers-panel-count {
  flex-shrink: 0;
  font-size: 14px;
  line-height: 20px;
  color: var(--body-text-subtle);
  text-align: right;
  white-space: nowrap;
}

/* Stat cards */
.stat-row {
  display: flex;
  gap: 24px;
  overflow-x: auto;
  padding-bottom: 2px;
}
.stat-card {
  display: flex;
  align-items: flex-end;
  gap: 6px;
  flex: 1 1 0;
  min-width: 200px;
  padding: 24px;
  border: 1px solid transparent;
  border-radius: 12px;
  background:
    linear-gradient(#ffffff, #ffffff) padding-box,
    linear-gradient(180deg, #ffd282 0%, #db5b6b 100%) border-box;
}
.stat-data {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 12px;
  min-width: 0;
}
.stat-label {
  margin: 0;
  font-size: 14px;
  color: var(--body-text);
}
.stat-value {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  line-height: 32px;
  color: var(--heading);
}
.stat-delta {
  display: flex;
  align-items: center;
  gap: 4px;
}
.delta-icon {
  width: 10px;
  height: auto;
}
.delta-value {
  font-size: 14px;
  font-weight: 500;
  color: var(--success);
}
.delta-caption {
  font-size: 14px;
  color: var(--body-text);
}
.stat-icon-wrap {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
  border-radius: 6px;
  flex-shrink: 0;
}

/* Widget grid (2x2), fixed at two columns regardless of available width. */
.widget-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
}

.widget-card {
  display: flex;
  flex-direction: column;
  gap: 24px;
  height: 462px;
  padding: 20px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 16px;
  min-width: 0;
}
.card-header {
  padding-bottom: 16px;
  border-bottom: 1px solid var(--border-light);
  flex-shrink: 0;
}
.card-header h2 {
  margin: 0 0 4px;
  font-size: 18px;
  font-weight: 600;
  color: var(--heading);
}
.card-header p {
  margin: 0;
  font-size: 14px;
  color: var(--heading);
}
.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 16px;
  border-top: 1px solid var(--border-light);
  flex-shrink: 0;
}
.card-footer span {
  font-size: 14px;
  color: var(--body-text);
}
.see-details {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  font-weight: 500;
  color: var(--brand);
  text-decoration: none;
}
.see-details img {
  width: 10.67px;
  height: 6px;
  transform: rotate(90deg);
}

:deep(.apexcharts-tooltip) {
  border: none !important;
  box-shadow: none !important;
  background: transparent !important;
}
:deep(.chart-tooltip) {
  padding: 8px 12px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(16, 24, 40, 0.12);
}
:deep(.chart-tooltip-value) {
  font-size: 14px;
  font-weight: 600;
  color: var(--heading);
}
:deep(.chart-tooltip-label) {
  margin-top: 2px;
  font-size: 12px;
  color: var(--body-text);
}

/* Bar / area charts (consumption analysis, preorder volume trend) */
.bar-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}

/* Radial bar chart (other status funnel) */
.radial-chart {
  position: relative;
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
  min-height: 0;
}

.radial-tooltip {
  position: absolute;
  transform: translate(-50%, calc(-100% - 10px));
  pointer-events: none;
  white-space: nowrap;
  z-index: 5;
}

.widget-legend {
  display: flex;
  justify-content: center;
  gap: 16px;
  flex-shrink: 0;
}
.legend-item {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  color: var(--body-text);
}
.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

/* Donut card (pickup method split) */
.donut-card {
  align-items: center;
  justify-content: space-between;
}
.donut-card .card-header,
.donut-card .card-footer {
  width: 100%;
}
.donut-chart {
  position: relative;
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
  width: 209px;
  height: 209px;
  margin: 0 auto;
}
.donut-label {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  text-align: center;
}
.donut-percent {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  line-height: 30px;
  color: var(--heading);
}
.donut-caption {
  margin: 0;
  font-size: 16px;
  color: var(--body-text);
}

/* ---------- Responsive ---------- */
@media (max-width: 1100px) {
  .widget-card {
    height: auto;
  }
}
@media (max-width: 720px) {
  .dashboard {
    flex-direction: column;
  }
  .dashboard-content {
    padding: 20px;
  }
  .header-section {
    flex-wrap: wrap;
  }
  .widget-grid {
    grid-template-columns: 1fr;
  }
  .dashboard.layers-panel-open .dashboard-main {
    margin-right: 0;
  }
  .layers-panel {
    top: 0;
    width: 100%;
  }
}
</style>
