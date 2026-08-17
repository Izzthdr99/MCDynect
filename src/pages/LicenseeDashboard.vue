<script setup>
/**
 * LicenseeDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1264-5217
 *
 * Licensee module landing dashboard: sidebar + topnav shell around license
 * compliance/fees/active-license stats, renewal forecasting, status breakdown,
 * fee collection, outstanding fees by licensee, license duration distribution,
 * application pipeline, performance vs fee, and termination rate widgets.
 * Reached from the workspace selector's "Licensee" module card. The topnav's
 * workspace switcher ("change role") is hidden here, same as Finance/Licensing/
 * Procurement/Outlet Manager/Inventory.
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
import iconStatComplianceRate from '../assets/dashboard/icons/icon-stat-compliance-rate.svg'
import iconStatOutstandingFees from '../assets/dashboard/icons/icon-stat-outstanding-fees.svg'
import iconStatActiveLicenses from '../assets/dashboard/icons/icon-stat-active-licenses.svg'

import iconGrid from '../assets/dashboard/sidebar/licensing/icon-grid.svg'
import iconChartLineUp from '../assets/dashboard/sidebar/operation/icon-chart-line-up.svg'
import iconCart from '../assets/dashboard/sidebar/licensing/icon-cart.svg'
import iconFileLines from '../assets/dashboard/sidebar/licensing/icon-file-lines.svg'
import iconClock from '../assets/dashboard/sidebar/operation/icon-clock.svg'
import iconRefresh from '../assets/dashboard/sidebar/licensing/icon-refresh.svg'
import iconTicket from '../assets/dashboard/sidebar/finance/icon-ticket.svg'
import iconLayersNav from '../assets/dashboard/icons/icon-layers.svg'
import iconMapPin from '../assets/dashboard/sidebar/licensee/icon-map-pin.svg'
import iconAdjustmentsHorizontal from '../assets/dashboard/sidebar/licensee/icon-adjustments-horizontal.svg'
import iconUndo from '../assets/dashboard/sidebar/licensee/icon-undo.svg'
import iconTools from '../assets/dashboard/sidebar/licensee/icon-tools.svg'
import iconList from '../assets/dashboard/sidebar/inventory/icon-list.svg'

import imgPatternBg from '../assets/dashboard/pattern-bg.jpg'

const emit = defineEmits(['back-to-workspace', 'sign-out'])

// Sidebar module list for the Licensee workspace.
const navItems = [
  { key: 'overview', icon: iconGrid, label: 'Overview', width: 15, height: 15 },
  { key: 'sales', icon: iconChartLineUp, label: 'Sales', width: 15, height: 14.58 },
  { key: 'order', icon: iconCart, label: 'Order', width: 13.33, height: 15 },
  { key: 'pre-order-tracking', icon: iconMapPin, label: 'Pre-Order Tracking', width: 11.67, height: 16.67 },
  { key: 'reports', icon: iconFileLines, label: 'Reports', width: 13.33, height: 16.67 },
  { key: 'lto', icon: iconClock, label: 'LTO', width: 16.67, height: 16.67 },
  { key: 'flexi-operation', icon: iconAdjustmentsHorizontal, label: 'Flexi Operation', width: 15, height: 15 },
  { key: 'renewal', icon: iconRefresh, label: 'Renewal', width: 13.33, height: 15 },
  { key: 'trade-in', icon: iconUndo, label: 'Trade-In', width: 16.67, height: 13.33 },
  { key: 'redemption-history', icon: iconTicket, label: 'Redemption History', width: 16.67, height: 11.67 },
  { key: 'mandatory-items', icon: iconList, label: 'Mandatory Items', width: 14.17, height: 11.39 },
  { key: 'equipment-replacement', icon: iconTools, label: 'Equipment Replacement', width: 15, height: 15 },
  { key: 'program', icon: iconLayersNav, label: 'Program', width: 13.33, height: 16.67 },
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
  { index: '1', category: 'License Renewals', count: '(8 Items)' },
  { index: '2', category: 'Outstanding Fee Follow-ups', count: '(5 Items)' },
  { index: '3', category: 'New License Applications', count: '(4 Items)' },
  { index: '4', category: 'Trade-In Requests', count: '(3 Items)' },
  { index: '5', category: 'Mandatory Item Reviews', count: '(6 Items)' },
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

// Stat cards
const statCards = [
  {
    key: 'license-compliance-rate',
    label: 'License Compliance Rate',
    value: '92%',
    delta: '0%',
    bg: '#fef2f2',
    icon: iconStatComplianceRate,
    iconW: 16.67,
    iconH: 16.67,
  },
  {
    key: 'outstanding-licensing-fees',
    label: 'Outstanding Licensing Fees',
    value: 'MYR 62,400',
    delta: '0%',
    bg: '#fffbeb',
    icon: iconStatOutstandingFees,
    iconW: 8.33,
    iconH: 16.67,
  },
  {
    key: 'active-licenses',
    label: 'Active Licenses',
    value: '210',
    delta: '0%',
    bg: '#f0fdf4',
    icon: iconStatActiveLicenses,
    iconW: 16.67,
    iconH: 16.67,
  },
]

// License renewal forecast — licenses expiring, projected next 3 months.
const renewalForecastMonths = ['Jul', 'Aug', 'Sep']
const renewalForecastChartOptions = {
  chart: { type: 'area', toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#2b7fff'],
  stroke: { curve: 'straight', width: 3 },
  fill: {
    type: 'gradient',
    gradient: { shadeIntensity: 1, opacityFrom: 0.35, opacityTo: 0, stops: [0, 90, 100] },
  },
  markers: { size: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: renewalForecastMonths,
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]} licenses</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const renewalForecastSeries = [{ name: 'Expiring', data: [8, 62, 92] }]

// License status overview — breakdown of licenses by status.
const licenseStatusBreakdown = [
  { label: 'Active', value: 38.4, color: '#05df72' },
  { label: 'Pending', value: 28.4, color: '#00b8db' },
  { label: 'Expired', value: 20.4, color: '#fdc700' },
  { label: 'Suspended', value: 12.8, color: '#ff6467' },
]
const licenseStatusChartOptions = {
  chart: { type: 'donut', toolbar: { show: false }, animations: { enabled: false } },
  labels: licenseStatusBreakdown.map((item) => item.label),
  colors: licenseStatusBreakdown.map((item) => item.color),
  dataLabels: { enabled: false },
  legend: { show: false },
  stroke: { show: false },
  plotOptions: { pie: { donut: { size: '75%', labels: { show: false } } } },
  tooltip: {
    custom: ({ series, seriesIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex]}%</div>
        <div class="chart-tooltip-label">${w.globals.labels[seriesIndex]}</div>
      </div>
    `,
  },
}
const licenseStatusSeries = licenseStatusBreakdown.map((item) => item.value)

// Annual fee collection — fees invoiced versus fees collected.
const feeCollectionMonths = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun']
const feeCollectionChartOptions = {
  chart: { type: 'area', toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#ff8904', '#1447e6'],
  stroke: { curve: 'smooth', width: [2, 2] },
  fill: {
    type: 'gradient',
    gradient: { shadeIntensity: 1, opacityFrom: 0.3, opacityTo: 0, stops: [0, 90, 100] },
  },
  markers: { size: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: feeCollectionMonths,
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    shared: true,
    custom: ({ series, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">RM ${series[0][dataPointIndex]}k invoiced &middot; RM ${series[1][dataPointIndex]}k collected</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const feeCollectionSeries = [
  { name: 'Invoiced', data: [42, 48, 45, 52, 58, 62] },
  { name: 'Collected', data: [38, 44, 42, 49, 54, 60] },
]

// Outstanding fees by licensee — overdue amount ranked by status.
const outstandingFeesByLicensee = [
  { name: 'Mr Churros Sri Uda', location: 'Kuala Lumpur', amount: 'RM 18,400' },
  { name: 'Mr Churros Taman Melati', location: 'Kuala Lumpur', amount: 'RM 14,200' },
  { name: 'Mr Churros Kota Damansara', location: 'Kuala Lumpur', amount: 'RM 9,800' },
  { name: 'Mr Churros Bukit Bintang', location: 'Kuala Lumpur', amount: 'RM 6,300' },
]

// License Duration Distribution — licensees grouped by tenure, ranked by count.
const licenseDurationBuckets = [
  { label: '0-1 year', value: 60, color: '#ff8a4c' },
  { label: '1-3 years', value: 95, color: '#51a2ff' },
  { label: '3-5 years', value: 62, color: '#05df72' },
  { label: '5 years +', value: 31, color: '#fdc700' },
]

// New License Applications — breakdown of applications by pipeline stage.
const applicationPipeline = [
  { label: 'Submitted', value: 40, color: '#1447e6' },
  { label: 'Under Review', value: 18, color: '#8ec5ff' },
  { label: 'Approved', value: 22, color: '#1447e6' },
  { label: 'Rejected', value: 5, color: '#8ec5ff' },
]
const applicationPipelineChartOptions = {
  chart: { type: 'bar', toolbar: { show: false }, animations: { enabled: false } },
  plotOptions: { bar: { columnWidth: '55%', borderRadius: 4, distributed: true } },
  colors: applicationPipeline.map((item) => item.color),
  dataLabels: {
    enabled: true,
    offsetY: -20,
    style: { fontSize: '12px', fontWeight: 600, colors: ['#101828'] },
  },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: applicationPipeline.map((item) => item.label),
    labels: { style: { colors: '#4a5565', fontSize: '13px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false, max: 48 },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]} applications</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const applicationPipelineSeries = [{ name: 'Applications', data: applicationPipeline.map((item) => item.value) }]

// Licensee Performance vs Fee — sales performance against fee paid.
const performanceVsFeeChartOptions = {
  chart: { type: 'scatter', toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#1447e6'],
  markers: { size: 5, strokeWidth: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { borderColor: '#f3f4f6' },
  xaxis: {
    title: { text: 'Fee Paid (RM k)', style: { color: '#6a7282', fontSize: '11px', fontWeight: 500 } },
    min: 0,
    max: 24,
    tickAmount: 6,
    labels: { style: { colors: '#6a7282', fontSize: '10px' } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: {
    min: 0,
    max: 40,
    labels: { style: { colors: '#6a7282', fontSize: '10px' } },
  },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">Performance ${series[seriesIndex][dataPointIndex][1]}</div>
        <div class="chart-tooltip-label">Fee paid RM ${series[seriesIndex][dataPointIndex][0]}k</div>
      </div>
    `,
  },
}
const performanceVsFeeSeries = [
  {
    name: 'Licensees',
    data: [
      [1, 8], [3, 12], [5, 14], [6, 16], [7, 20], [9, 18],
      [10, 24], [11, 20], [12, 29], [14, 27], [17, 28], [19, 33], [23, 40],
    ],
  },
]

// Termination/Non-Renewal Rate — yearly rate of licenses terminated or not renewed.
const terminationYears = ['2022', '2023', '2024', '2025', '2026']
const terminationRateChartOptions = {
  chart: { type: 'area', toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#d53f52'],
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
    categories: terminationYears,
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]}%</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const terminationRateSeries = [{ name: 'Termination rate', data: [9.2, 8.1, 6.8, 5.9, 4.7] }]
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

        <!-- License renewal forecast + License status overview -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>License renewal forecast</h2>
              <p>Licenses expiring, projected next 3 months</p>
            </div>
            <div class="line-chart-container">
              <VueApexCharts
                type="area"
                width="100%"
                height="100%"
                :options="renewalForecastChartOptions"
                :series="renewalForecastSeries"
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
              <h2>License status overview</h2>
              <p>Breakdown of licenses by status</p>
            </div>
            <div class="donut-chart">
              <VueApexCharts
                type="donut"
                width="100%"
                height="100%"
                :options="licenseStatusChartOptions"
                :series="licenseStatusSeries"
              />
              <div class="donut-label">
                <p class="donut-percent">{{ licenseStatusBreakdown[0].value }}%</p>
                <p class="donut-caption">{{ licenseStatusBreakdown[0].label }}</p>
              </div>
            </div>
            <div class="widget-legend">
              <div v-for="status in licenseStatusBreakdown" :key="status.label" class="legend-item">
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
        </div>

        <!-- Annual fee collection + Outstanding fees by licensee -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>Annual fee collection</h2>
              <p>Fees invoiced versus fees collected.</p>
            </div>
            <div class="line-chart-container">
              <VueApexCharts
                type="area"
                width="100%"
                height="100%"
                :options="feeCollectionChartOptions"
                :series="feeCollectionSeries"
              />
            </div>
            <div class="widget-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #ff8904"></span><span>Invoiced</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #1447e6"></span><span>Collected</span></div>
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
              <h2>Outstanding fees by licensee</h2>
              <p>Overdue amount ranked by status.</p>
            </div>
            <div class="fee-table">
              <div v-for="(licensee, i) in outstandingFeesByLicensee" :key="licensee.name" class="fee-table-row">
                <div class="fee-table-name">
                  <span class="fee-table-rank">{{ i + 1 }}.</span>
                  <div class="fee-table-name-helper">
                    <span class="fee-table-name-text">{{ licensee.name }}</span>
                    <span class="fee-table-location">{{ licensee.location }}</span>
                  </div>
                </div>
                <span class="fee-table-amount">{{ licensee.amount }}</span>
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
        </div>

        <!-- License Duration Distribution + New License Applications -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>License Duration Distribution</h2>
              <p>Licensees grouped by tenure, ranked by count</p>
            </div>
            <div class="duration-distribution">
              <div class="duration-bar">
                <span
                  v-for="bucket in licenseDurationBuckets"
                  :key="bucket.label"
                  class="duration-segment"
                  :style="{ flexGrow: bucket.value, background: bucket.color }"
                ></span>
              </div>
              <div class="duration-stats">
                <div v-for="bucket in licenseDurationBuckets" :key="bucket.label" class="duration-stat">
                  <p class="duration-stat-value">{{ bucket.value }}</p>
                  <div class="duration-stat-label">
                    <span class="legend-dot" :style="{ background: bucket.color }"></span>
                    <span>{{ bucket.label }}</span>
                  </div>
                </div>
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
              <h2>New License Applications</h2>
              <p>Breakdown of applications by pipeline stage.</p>
            </div>
            <div class="v-bar-chart-container">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="applicationPipelineChartOptions"
                :series="applicationPipelineSeries"
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

        <!-- Licensee Performance vs Fee + Termination/Non-Renewal Rate -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>Licensee Performance vs Fee</h2>
              <p>Sales performance against fee paid.</p>
            </div>
            <div class="scatter-chart-container">
              <VueApexCharts
                type="scatter"
                width="100%"
                height="100%"
                :options="performanceVsFeeChartOptions"
                :series="performanceVsFeeSeries"
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

          <div class="widget-card">
            <div class="card-header">
              <h2>Termination/ Non-Renewal Rate</h2>
              <p>Yearly rate of licenses terminated or not renewed.</p>
            </div>
            <div class="line-chart-container">
              <VueApexCharts
                type="area"
                width="100%"
                height="100%"
                :options="terminationRateChartOptions"
                :series="terminationRateSeries"
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
    rgba(249, 250, 251, 0.946) 95%,
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

/* Widget rows (pairs, matching the Figma "Bottom Row" frames) */
.widget-row {
  display: flex;
  gap: 24px;
}

.widget-card {
  display: flex;
  flex: 1;
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

/* Line / area charts (renewal forecast, fee collection, termination rate) */
.line-chart-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}

/* Vertical bar chart (new license applications) */
.v-bar-chart-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: center;
  min-height: 0;
}

/* Scatter chart (licensee performance vs fee) */
.scatter-chart-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: center;
  min-height: 0;
}

.widget-legend {
  display: flex;
  flex-wrap: wrap;
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
  flex-shrink: 0;
}

/* Donut card (license status overview) */
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

/* Outstanding fees by licensee table */
.fee-table {
  display: flex;
  flex: 1;
  flex-direction: column;
  min-height: 0;
}
.fee-table-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  height: 56px;
  border-bottom: 1px solid var(--border-light);
  flex-shrink: 0;
}
.fee-table-row:last-child {
  border-bottom: none;
}
.fee-table-name {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  min-width: 0;
}
.fee-table-rank {
  flex-shrink: 0;
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
}
.fee-table-name-helper {
  display: flex;
  flex-direction: column;
  gap: 1px;
  min-width: 0;
}
.fee-table-name-text {
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
  white-space: nowrap;
}
.fee-table-location {
  font-size: 14px;
  color: var(--body-text);
}
.fee-table-amount {
  flex-shrink: 0;
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
  white-space: nowrap;
}

/* License Duration Distribution (segmented bar + stat breakdown) */
.duration-distribution {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: center;
  gap: 24px;
  min-height: 0;
}
.duration-bar {
  display: flex;
  width: 100%;
  height: 20px;
  border-radius: 4px;
  overflow: hidden;
}
.duration-segment {
  height: 100%;
  min-width: 4px;
}
.duration-stats {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.duration-stat {
  display: flex;
  flex: 1;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
.duration-stat-value {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  line-height: 28px;
  color: var(--heading);
}
.duration-stat-label {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
  white-space: nowrap;
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
  .stat-row {
    flex-direction: column;
  }
  .widget-row {
    flex-direction: column;
  }
  .donut-card {
    width: 100%;
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
