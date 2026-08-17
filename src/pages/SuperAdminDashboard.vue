<script setup>
/**
 * SuperAdminDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=926-12145
 *
 * SuperAdmin module landing dashboard: sidebar + topnav shell around sales
 * KPI cards, trend charts, and revenue ranking tables. Reached from the
 * workspace selector's "Super Admin" module card.
 */
import { onBeforeUnmount, onMounted, ref } from 'vue'
import VueApexCharts from 'vue3-apexcharts'
import DashboardSidebar from '../components/DashboardSidebar.vue'
import DashboardTopnav from '../components/DashboardTopnav.vue'

import iconDashboard from '../assets/dashboard/sidebar/icon-dashboard.svg'
import iconStaffRoles from '../assets/dashboard/sidebar/icon-staff-roles.svg'
import iconSettings from '../assets/dashboard/sidebar/icon-settings.svg'
import iconAngleDown from '../assets/dashboard/icons/icon-angle-down.svg'
import iconCalendarMonth from '../assets/dashboard/icons/icon-calendar-month.svg'
import iconLayers from '../assets/dashboard/icons/icon-layers.svg'
import iconMinus from '../assets/dashboard/icons/icon-minus.svg'
import iconStatSales from '../assets/dashboard/icons/icon-stat-sales.svg'
import iconStatMtd from '../assets/dashboard/icons/icon-stat-mtd.svg'
import iconStatKg from '../assets/dashboard/icons/icon-stat-kg.svg'
import iconStatWaste from '../assets/dashboard/icons/icon-stat-waste.svg'
import iconStatAvg from '../assets/dashboard/icons/icon-stat-avg.svg'
import iconChevronDetails from '../assets/dashboard/icons/icon-chevron-see-details.svg'
import iconTrophyGold from '../assets/dashboard/icons/icon-trophy-gold.svg'
import iconTrophySilver from '../assets/dashboard/icons/icon-trophy-silver.svg'
import iconTrophyBronze from '../assets/dashboard/icons/icon-trophy-bronze.svg'
import imgPatternBg from '../assets/dashboard/pattern-bg.jpg'

const emit = defineEmits(['back-to-workspace', 'sign-out'])

const navItems = [
  { key: 'dashboard', icon: iconDashboard, label: 'Dashboard', width: 15, height: 15 },
  { key: 'staff-roles', icon: iconStaffRoles, label: 'Staff & Roles', width: 16.67, height: 13.33 },
  { key: 'system-settings', icon: iconSettings, label: 'System Settings', width: 16.67, height: 16.67 },
]

const activeNavItem = ref('dashboard')
const sidebarCollapsed = ref(false)

function handleNavigate(key) {
  // Only Dashboard is built; other routes are no-ops for now.
  if (key === 'dashboard') activeNavItem.value = key
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
  { index: '1', category: 'Corporate Finance', count: '(110 Items)' },
  { index: '2', category: 'Financial Conrollership', count: '(03 Items)' },
  { index: '3', category: 'Finance Operations', count: '(01 Items)' },
  { index: '4', category: 'Treasury & Acoounting', count: '(01 Items)' },
  { index: '5', category: 'Ledger Management', count: '(01 Items)' },
  { index: '5', category: 'Invoice Tracking', count: '(15 Items)' },
  { index: '5', category: 'Expense Reports', count: '(23 Items)' },
  { index: '5', category: 'Budget Planning', count: '(9 Items)' },
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

// Icon width/height match each SVG's own exported Figma bounding box —
// forcing them all into one square (as an earlier pass did) stretches
// every icon that isn't already square.
const statCards = [
  { key: 'today-sales', label: "Today's Sales", value: 'MYR 24,580', delta: '0%', bg: '#eef6ff', icon: iconStatSales, iconW: 11.67, iconH: 16.67 },
  { key: 'mtd-sales', label: 'MTD Sales', value: 'MYR 612,900', delta: '0%', bg: '#f0fdf4', icon: iconStatMtd, iconW: 15, iconH: 15 },
  { key: 'total-kg', label: 'Total KG Sold', value: '18,240 kg', delta: '0%', bg: '#eef2ff', icon: iconStatKg, iconW: 13.33, iconH: 16.67 },
  { key: 'unsold-waste', label: 'Unsold / Waste', value: '342 sets', delta: '10%', bg: '#fef2f2', icon: iconStatWaste, iconW: 13.33, iconH: 16.67 },
  { key: 'avg-sales-kg', label: 'Avg. Sales / KG', value: 'MYR 33.60', delta: '-5%', bg: '#faf5ff', icon: iconStatAvg, iconW: 16.67, iconH: 13.33 },
]

function formatMyr(value) {
  return `MYR ${Math.round(value).toLocaleString('en-MY')}`
}

const salesTrend = [
  { day: 'Jan 21', value: 18400 },
  { day: 'Jan 22', value: 21900 },
  { day: 'Jan 23', value: 19600 },
  { day: 'Jan 24', value: 27300 },
  { day: 'Jan 25', value: 20100 },
  { day: 'Jan 26', value: 25800 },
  { day: 'Jan 27', value: 22400 },
]

const salesTrendChartOptions = {
  chart: {
    type: 'area',
    toolbar: { show: false },
    zoom: { enabled: false },
    animations: { enabled: false },
  },
  colors: ['#155DFC'],
  fill: {
    type: 'gradient',
    colors: ['#1447E6'],
    gradient: {
      shadeIntensity: 1,
      opacityFrom: 0.4,
      opacityTo: 0,
      stops: [0, 100],
    },
  },
  stroke: { curve: 'straight', width: 2 },
  dataLabels: { enabled: false },
  grid: { show: false },
  xaxis: {
    categories: salesTrend.map((point) => point.day),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const salesTrendSeries = [{ name: 'Sales', data: salesTrend.map((point) => point.value) }]

const cumulativeSales = [
  { day: 'Jan 21', value: 15200 },
  { day: 'Jan 22', value: 19800 },
  { day: 'Jan 23', value: 17500 },
  { day: 'Jan 24', value: 21300 },
  { day: 'Jan 25', value: 16900 },
  { day: 'Jan 26', value: 24600 },
  { day: 'Jan 27', value: 23100 },
]

const cumulativeSalesChartOptions = {
  chart: {
    type: 'line',
    toolbar: { show: false },
    zoom: { enabled: false },
    animations: { enabled: false },
  },
  colors: ['#51A2FF'],
  stroke: { curve: 'smooth', width: 2 },
  dataLabels: { enabled: false },
  grid: { show: false },
  xaxis: {
    categories: cumulativeSales.map((point) => point.day),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const cumulativeSalesSeries = [{ name: 'Sales', data: cumulativeSales.map((point) => point.value) }]

const salesByOutlet = [
  { rank: 1, name: 'Mr Churros Bandar Sri Uda', location: 'Kuala Lumpur', amount: 'RM 34,000', trophy: iconTrophyGold },
  { rank: 2, name: 'Mr Churros Bandar Sri Uda II', location: 'Kuala Lumpur', amount: 'RM 30,000', trophy: iconTrophySilver },
  { rank: 3, name: 'Mr Churros Taman Melati', location: 'Kuala Lumpur', amount: 'RM 29,000', trophy: iconTrophyBronze },
  { rank: 4, name: 'Mr Churros Kota Damansara', location: 'Kuala Lumpur', amount: 'RM 14,000', trophy: null },
  { rank: 5, name: 'Mr Churros Bukit Bintang', location: 'Kuala Lumpur', amount: 'RM 10,000', trophy: null },
]

const salesByLicensee = [
  { key: 'licensee-1', name: 'Mr Churros Klang Valley', amount: 76500, color: '#00a6f4' },
  { key: 'licensee-2', name: 'Mr Churros Johor Bahru', amount: 152000, color: '#ff8a4c' },
  { key: 'licensee-3', name: 'Mr Churros Penang', amount: 113000, color: '#05df72' },
  { key: 'licensee-4', name: 'Mr Churros Ipoh', amount: 34000, color: '#ff6467' },
  { key: 'licensee-5', name: 'Mr Churros Melaka', amount: 63500, color: '#c27aff' },
  { key: 'licensee-6', name: 'Mr Churros Kuching', amount: 100500, color: '#fdc700' },
]

const salesByLicenseeChartOptions = {
  chart: {
    type: 'bar',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  plotOptions: {
    bar: {
      horizontal: true,
      distributed: true,
      borderRadius: 4,
      barHeight: '60%',
    },
  },
  colors: salesByLicensee.map((licensee) => licensee.color),
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: salesByLicensee.map((licensee) => licensee.name),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { labels: { show: false } },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const salesByLicenseeSeries = [{ name: 'Revenue', data: salesByLicensee.map((licensee) => licensee.amount) }]

const salesByChannel = [
  { label: 'Cash', value: 82.4, color: '#ff8a4c' },
  { label: 'Cash Equivalent', value: 17.6, color: '#00bcff' },
]

const donutChartOptions = {
  chart: {
    type: 'donut',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  labels: salesByChannel.map((channel) => channel.label),
  colors: salesByChannel.map((channel) => channel.color),
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

const donutSeries = salesByChannel.map((channel) => channel.value)

const salesByState = [
  { rank: 1, name: 'Selangor', amount: 'RM 51,500' },
  { rank: 2, name: 'Johor', amount: 'RM 50,000' },
  { rank: 3, name: 'Penang', amount: 'RM 48,450' },
  { rank: 4, name: 'Kedah', amount: 'RM 47,000' },
  { rank: 5, name: 'Perak', amount: 'RM 45,500' },
  { rank: 6, name: 'Kelantan', amount: 'RM 44,000' },
  { rank: 7, name: 'Terengganu', amount: 'RM 42,600' },
  { rank: 8, name: 'Pahang', amount: 'RM 41,100' },
  { rank: 9, name: 'Sabah', amount: 'RM 39,750' },
  { rank: 10, name: 'Sarawak', amount: 'RM 38,200' },
  { rank: 11, name: 'Negeri Sembilan', amount: 'RM 36,500' },
  { rank: 12, name: 'Melaka', amount: 'RM 34,000' },
  { rank: 13, name: 'Perlis', amount: 'RM 34,000' },
]
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
        @toggle-sidebar="sidebarCollapsed = !sidebarCollapsed"
        @switch-workspace="emit('back-to-workspace')"
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
          <div class="chart-card">
            <div class="card-header">
              <h2>Sales Trend (Daily</h2>
              <p>Daily revenue trend across all outlets.</p>
            </div>
            <div class="line-chart">
              <div class="line-chart-canvas">
                <VueApexCharts
                  type="area"
                  width="100%"
                  height="100%"
                  :options="salesTrendChartOptions"
                  :series="salesTrendSeries"
                />
              </div>
              <div class="chart-x-axis">
                <span v-for="point in salesTrend" :key="point.day">{{ point.day }}</span>
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

          <div class="chart-card">
            <div class="card-header">
              <h2>Sales By Outlet</h2>
              <p>Top-performing outlets ranked by revenue this period.</p>
            </div>
            <ul class="ranked-list">
              <li v-for="outlet in salesByOutlet" :key="outlet.rank" class="ranked-row">
                <div class="ranked-name">
                  <span class="ranked-index">{{ outlet.rank }}.</span>
                  <div class="ranked-name-helper">
                    <p class="ranked-title">{{ outlet.name }}</p>
                    <p class="ranked-subtitle">{{ outlet.location }}</p>
                  </div>
                </div>
                <div class="ranked-amount">
                  <img v-if="outlet.trophy" class="trophy-icon" :src="outlet.trophy" alt="" />
                  <span>{{ outlet.amount }}</span>
                </div>
              </li>
            </ul>
            <div class="card-footer">
              <span>Last Updated : Today, 2:45PM</span>
              <a href="#" class="see-details">
                See Details
                <img :src="iconChevronDetails" alt="" />
              </a>
            </div>
          </div>
        </div>

        <div class="widget-grid">
          <div class="chart-card">
            <div class="card-header">
              <h2>Cumulative Sales</h2>
              <p>Running total of sales tracked against the previous period.</p>
            </div>
            <div class="line-chart">
              <div class="line-chart-canvas">
                <VueApexCharts
                  type="line"
                  width="100%"
                  height="100%"
                  :options="cumulativeSalesChartOptions"
                  :series="cumulativeSalesSeries"
                />
              </div>
              <div class="chart-x-axis">
                <span v-for="point in cumulativeSales" :key="point.day">{{ point.day }}</span>
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

          <div class="chart-card">
            <div class="card-header">
              <h2>Sales By Licensee</h2>
              <p>Rank licensee revenue from highest to lowest.</p>
            </div>
            <div class="bar-chart">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="salesByLicenseeChartOptions"
                :series="salesByLicenseeSeries"
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

        <div class="widget-grid">
          <div class="donut-card">
            <div class="card-header">
              <h2>Sales By Channel</h2>
              <p>Dine-in, delivery, takeaway.</p>
            </div>
            <div class="donut-chart">
              <VueApexCharts
                type="donut"
                width="100%"
                height="100%"
                :options="donutChartOptions"
                :series="donutSeries"
              />
              <div class="donut-label">
                <p class="donut-percent">{{ salesByChannel[0].value }}%</p>
                <p class="donut-caption">{{ salesByChannel[0].label }}</p>
              </div>
            </div>
            <div class="donut-legend">
              <div v-for="channel in salesByChannel" :key="channel.label" class="legend-item">
                <span class="legend-dot" :style="{ background: channel.color }"></span>
                <span>{{ channel.label }}</span>
              </div>
            </div>
          </div>

          <div class="chart-card">
            <div class="card-header">
              <h2>Sales By State</h2>
              <p>State sales performance ranked by revenue.</p>
            </div>
            <ul class="ranked-list scrollable">
              <li v-for="state in salesByState" :key="state.rank" class="ranked-row">
                <div class="ranked-name">
                  <span class="ranked-index">{{ state.rank }}.</span>
                  <p class="ranked-title">{{ state.name }}</p>
                </div>
                <div class="ranked-amount">
                  <span>{{ state.amount }}</span>
                </div>
              </li>
            </ul>
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
  /* Sidebar is position: fixed (out of flex flow) and the pending-activity
     panel is too, so their widths have to be reserved here manually
     instead of the flexbox row sizing them. */
  margin-left: 251px;
  margin-right: 0;
  transition: margin-left 220ms ease, margin-right 220ms ease;
}

/* Decorative pattern behind the topnav + content, fading to solid white
   about halfway down so it never competes with the widgets below the fold. */
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
  /* Topnav is position: fixed (out of flow), so its height is reserved
     here manually instead of flexbox stacking it above this content. */
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
/* Icon width/height match each SVG's own native aspect ratio — the angle-down
   chevron (10.67x6) and calendar-month glyph (12x12) aren't interchangeable,
   so each gets its own size instead of being squashed into one shared box. */
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
/* stat-icon-wrap img sizing is set inline per-icon (see statCards) since
   each exported SVG has its own native aspect ratio. */

/* Widget grid (two-up chart rows). auto-fit/minmax reflows off the grid's
   own rendered width, so it collapses to one column whenever the layers
   panel (or a narrow viewport) leaves too little room for two — no
   viewport-only media query needed to react to the panel opening. */
.widget-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 24px;
}

.chart-card {
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

/* Line charts */
.line-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: flex-end;
  min-height: 0;
}
.line-chart-canvas {
  flex: 1;
  min-height: 0;
  width: 100%;
}

/* Shared ApexCharts tooltip styling. ApexCharts injects this markup directly
   into the chart's DOM at runtime, bypassing Vue's compiler — :deep() is
   required for scoped styles to reach it. */
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
.chart-x-axis {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 16px;
  flex-shrink: 0;
}
.chart-x-axis span {
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
}

/* Ranked lists (outlet / state tables) */
.ranked-list {
  display: flex;
  flex: 1;
  flex-direction: column;
  min-height: 0;
  margin: 0;
  padding: 0;
  list-style: none;
}
.ranked-list.scrollable {
  overflow-y: auto;
}
.ranked-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  height: 56px;
  flex-shrink: 0;
  border-bottom: 1px solid var(--border-light);
}
.ranked-row:last-child {
  border-bottom: none;
}
.ranked-name {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  min-width: 0;
}
.ranked-index {
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
  flex-shrink: 0;
}
.ranked-name-helper {
  display: flex;
  flex-direction: column;
  gap: 1px;
  min-width: 0;
}
.ranked-title {
  margin: 0;
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.ranked-subtitle {
  margin: 0;
  font-size: 14px;
  color: var(--body-text);
}
.ranked-amount {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-shrink: 0;
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
}
.trophy-icon {
  width: 16px;
  height: 16px;
}

/* Bar chart (licensee) */
.bar-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}

/* Donut card */
.donut-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  height: 462px;
  padding: 24px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: 0px 1px 0.25px rgba(29, 41, 61, 0.02);
}
.donut-card .card-header {
  width: 100%;
}
.donut-chart {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 209px;
  height: 209px;
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
.donut-legend {
  display: flex;
  gap: 16px;
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

/* ---------- Responsive ---------- */
@media (max-width: 1100px) {
  .chart-card,
  .donut-card {
    height: auto;
  }
  .line-chart-canvas {
    height: 220px;
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
  .dashboard.layers-panel-open .dashboard-main {
    margin-right: 0;
  }
  .layers-panel {
    top: 0;
    width: 100%;
  }
}
</style>
