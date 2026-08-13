<script setup>
/**
 * InventoryDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1407-27392
 *
 * Inventory module landing dashboard: sidebar + topnav shell around SKU/reorder/
 * fulfillment stats and forecasting, turnover, ABC analysis, stockout, supplier
 * lead time, inventory value, slow-moving stock, and demand vs supply widgets.
 * Reached from the workspace selector's "Inventory" module card. The topnav's
 * workspace switcher ("change role") is hidden here, same as Finance/Licensing/
 * Procurement/Outlet Manager.
 */
import { onBeforeUnmount, onMounted, ref } from 'vue'
import VueApexCharts from 'vue3-apexcharts'
import DashboardSidebar from '../components/DashboardSidebar.vue'
import DashboardTopnav from '../components/DashboardTopnav.vue'

import iconAngleDown from '../assets/dashboard/icons/icon-angle-down.svg'
import iconCalendarMonth from '../assets/dashboard/icons/icon-calendar-month.svg'
import iconLayers from '../assets/dashboard/icons/icon-layers.svg'
import iconMinus from '../assets/dashboard/icons/icon-minus.svg'
import iconTrendUp from '../assets/dashboard/icons/icon-trend-up.svg'
import iconChevronDetails from '../assets/dashboard/icons/icon-chevron-see-details.svg'
import iconStatTotalSkus from '../assets/dashboard/icons/icon-stat-total-skus.svg'
import iconStatReorderPoint from '../assets/dashboard/icons/icon-stat-reorder-point.svg'
import iconStatOrderFulfillment from '../assets/dashboard/icons/icon-stat-order-fulfillment.svg'

import iconGrid from '../assets/dashboard/sidebar/licensing/icon-grid.svg'
import iconFileLines from '../assets/dashboard/sidebar/licensing/icon-file-lines.svg'
import iconBarChart from '../assets/dashboard/sidebar/licensing/icon-bar-chart.svg'
import iconRefresh from '../assets/dashboard/sidebar/licensing/icon-refresh.svg'
import iconCart from '../assets/dashboard/sidebar/licensing/icon-cart.svg'
import iconMapPinAlt from '../assets/dashboard/sidebar/licensing/icon-map-pin-alt.svg'
import iconArrowsRepeat from '../assets/dashboard/sidebar/licensing/icon-arrows-repeat.svg'
import iconShoppingBag from '../assets/dashboard/sidebar/operation/icon-shopping-bag.svg'
import iconList from '../assets/dashboard/sidebar/inventory/icon-list.svg'

import imgPatternBg from '../assets/dashboard/pattern-bg.jpg'

const emit = defineEmits(['back-to-workspace', 'sign-out'])

// Sidebar module list for the Inventory workspace. RDC Inventory, Orders, and
// Order Records show a chevron affordance but don't open sub-navigation yet,
// matching the no-op pattern used by the other module sidebars.
const navItems = [
  { key: 'overview', icon: iconGrid, label: 'Overview', width: 15, height: 15 },
  { key: 'item-master-data', icon: iconFileLines, label: 'Item Master Data', width: 13.33, height: 16.67 },
  { key: 'item-management', icon: iconBarChart, label: 'Item Management', width: 16.67, height: 15 },
  { key: 'safe-stock-level', icon: iconRefresh, label: 'Safe Stock Level', width: 16.67, height: 16.67 },
  { key: 'reorder', icon: iconCart, label: 'Reorder', width: 13.33, height: 15 },
  { key: 'order-request', icon: iconShoppingBag, label: 'Order Request', width: 13.33, height: 16.67 },
  { key: 'rdc-inventory', icon: iconMapPinAlt, label: 'RDC Inventory', width: 13.33, height: 16.67, chevron: true },
  { key: 'orders', icon: iconList, label: 'Orders', width: 16.67, height: 11.67, chevron: true },
  { key: 'order-records', icon: iconArrowsRepeat, label: 'Order Records', width: 13.33, height: 15, chevron: true },
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
  { index: '1', category: 'Reorder Approvals', count: '(6 Items)' },
  { index: '2', category: 'Safe Stock Level Adjustments', count: '(4 Items)' },
  { index: '3', category: 'Order Request Reviews', count: '(9 Items)' },
  { index: '4', category: 'RDC Inventory Transfers', count: '(3 Items)' },
  { index: '5', category: 'Slow-Moving Stock Follow-ups', count: '(5 Items)' },
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
    key: 'total-skus',
    label: 'Total SKUs',
    value: '1842',
    delta: '+12',
    trend: 'up',
    bg: '#fef2f2',
    icon: iconStatTotalSkus,
    iconW: 13.33,
    iconH: 16.67,
  },
  {
    key: 'reorder-point-status',
    label: 'Reorder Point Status',
    value: '68%',
    delta: '0%',
    trend: 'flat',
    bg: '#fffbeb',
    icon: iconStatReorderPoint,
    iconW: 11.67,
    iconH: 16.67,
  },
  {
    key: 'order-fulfillment-rate',
    label: 'Order Fulfillment Rate',
    value: '94%',
    delta: '+5%',
    trend: 'up',
    bg: '#f0fdf4',
    icon: iconStatOrderFulfillment,
    iconW: 17,
    iconH: 18.67,
  },
]

// Inventory Forecasting — stockout count against a 3-month forecast.
const forecastCategories = ['Aug', 'Sep', 'Oct']
const inventoryForecastingChartOptions = {
  chart: { type: 'area', toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#2b7fff', '#ff8904'],
  stroke: { curve: 'straight', width: [3, 3] },
  fill: {
    type: 'gradient',
    gradient: { shadeIntensity: 1, opacityFrom: 0.35, opacityTo: 0, stops: [0, 90, 100] },
  },
  markers: { size: 6, strokeWidth: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: forecastCategories,
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    shared: true,
    custom: ({ series, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[0][dataPointIndex] ?? series[1][dataPointIndex]} stockouts</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
// Actual (Aug -> Sep) and Forecast (Sep -> Oct) share the Sep data point so the
// two colored segments connect into one continuous rising line, matching the
// Figma design's single split-color trend line rather than two parallel lines.
const inventoryForecastingSeries = [
  { name: 'Actual', data: [6, 11, null] },
  { name: 'Forecast', data: [null, 11, 17] },
]

// Stock Level Trend — quantity on hand across the last 30 days (dips early,
// recovers mid-month, then declines into W4).
const stockLevelByWeek = [
  { week: 'W1', value: 1200 },
  { week: 'W2', value: 820 },
  { week: 'W3', value: 1080 },
  { week: 'W4', value: 640 },
]
const stockLevelTrendChartOptions = {
  chart: { type: 'area', toolbar: { show: false }, animations: { enabled: false } },
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
    categories: stockLevelByWeek.map((point) => point.week),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex].toLocaleString('en-MY')} units</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const stockLevelTrendSeries = [{ name: 'Quantity on hand', data: stockLevelByWeek.map((point) => point.value) }]

// Inventory Turnover Ratio — turnover rate by category this month.
const turnoverByCategory = [
  { label: 'Beverages', value: 6.2 },
  { label: 'Snacks', value: 4.8 },
  { label: 'Frozen', value: 3.1 },
  { label: 'Dry Goods', value: 2.4 },
]
const turnoverChartOptions = {
  chart: { type: 'bar', toolbar: { show: false }, animations: { enabled: false } },
  plotOptions: { bar: { horizontal: true, borderRadius: 4, barHeight: '55%' } },
  colors: ['#1447e6'],
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: turnoverByCategory.map((item) => item.label),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } } },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]}x</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const turnoverSeries = [{ name: 'Turnover', data: turnoverByCategory.map((item) => item.value) }]

// ABC Analysis — items ranked by value contribution, classified into tiers.
const abcItems = [
  { label: 'Item 1', value: 420 },
  { label: 'Item 2', value: 340 },
  { label: 'Item 3', value: 268 },
  { label: 'Item 4', value: 155 },
  { label: 'Item 5', value: 118 },
  { label: 'Item 6', value: 77 },
  { label: 'Item 7', value: 49 },
  { label: 'Item 8', value: 20 },
]
const abcTotal = abcItems.reduce((sum, item) => sum + item.value, 0)
let abcRunningTotal = 0
const abcCumulativePercent = abcItems.map((item) => {
  abcRunningTotal += item.value
  return Number(((abcRunningTotal / abcTotal) * 100).toFixed(1))
})
const abcAnalysisChartOptions = {
  chart: { toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#1447e6', '#2b7fff', '#8ec5ff', '#911b2d'],
  stroke: { width: [0, 0, 0, 2], curve: 'smooth' },
  plotOptions: { bar: { columnWidth: '55%', borderRadius: 4 } },
  markers: { size: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: abcItems.map((item) => item.label),
    labels: { show: false },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: [
    { show: false },
    { show: false, opposite: true, min: 0, max: 100 },
  ],
  tooltip: {
    shared: true,
    custom: ({ series, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">RM ${abcItems[dataPointIndex].value} &middot; ${series[3][dataPointIndex]}% cumulative</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const abcAnalysisSeries = [
  { name: 'Tier A', type: 'column', data: abcItems.map((item, i) => (i < 3 ? item.value : null)) },
  { name: 'Tier B', type: 'column', data: abcItems.map((item, i) => (i >= 3 && i < 6 ? item.value : null)) },
  { name: 'Tier C', type: 'column', data: abcItems.map((item, i) => (i >= 6 ? item.value : null)) },
  { name: 'Cumulative %', type: 'line', data: abcCumulativePercent },
]

// Stockout Frequency — number of stockout events by category.
const stockoutByCategory = [
  { label: 'Frozen', value: 42 },
  { label: 'Dry Goods', value: 31 },
  { label: 'Snacks', value: 23 },
  { label: 'Beverages', value: 15 },
]
const stockoutChartOptions = {
  chart: { type: 'bar', toolbar: { show: false }, animations: { enabled: false } },
  plotOptions: { bar: { horizontal: true, borderRadius: 4, barHeight: '55%' } },
  colors: ['#af2136'],
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: stockoutByCategory.map((item) => item.label),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } } },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]} stockouts</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const stockoutSeries = [{ name: 'Stockouts', data: stockoutByCategory.map((item) => item.value) }]

// Supplier Lead Time — actual versus promised delivery time by supplier.
// Custom dumbbell/range track (no ApexCharts range-bar tooltip support), built
// as absolutely-positioned dots over a horizontal scale, same approach used by
// the Procurement radial chart's manual hover tracking.
const supplierLeadTimes = [
  { name: 'Supplier A', promised: 7, actual: 9.2, delta: '+2.2', trend: 'over' },
  { name: 'Supplier B', promised: 6, actual: 7.1, delta: '+1.1', trend: 'over' },
  { name: 'Supplier C', promised: 5, actual: 7.6, delta: '+2.6', trend: 'over' },
  { name: 'Supplier D', promised: 10, actual: 9.2, delta: '-0.8', trend: 'under' },
]
const supplierLeadTimeScaleMax = 12
function leadTimePercent(value) {
  return (value / supplierLeadTimeScaleMax) * 100
}

// Inventory Value by Category — distribution of inventory value across categories.
const inventoryValueByCategory = [
  { label: 'Frozen', value: 35, color: '#51a2ff' },
  { label: 'Dry Goods', value: 27, color: '#ff8904' },
  { label: 'Snacks', value: 22, color: '#00a63e' },
  { label: 'Beverages', value: 16, color: '#911b2d' },
]
const inventoryValueChartOptions = {
  chart: { type: 'donut', toolbar: { show: false }, animations: { enabled: false } },
  labels: inventoryValueByCategory.map((item) => item.label),
  colors: inventoryValueByCategory.map((item) => item.color),
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
const inventoryValueSeries = inventoryValueByCategory.map((item) => item.value)
const inventoryTotalValueLabel = 'RM2.67M'

// Slow-Moving/Dead Stock — items with low or no movement over the last 90 days,
// as a 4x4 warehouse heatmap. Cell colors follow the brand red scale.
const slowMovingWarehouses = ['WH1', 'WH2', 'WH3', 'WH4']
const slowMovingHeatmapRows = [
  {
    label: 'Frozen',
    cells: [
      { value: 80, bg: '#721623', text: '#ffffff' },
      { value: 20, bg: '#e47081', text: '#ffffff' },
      { value: 10, bg: '#f9dfe3', text: '#0f172b' },
      { value: 5, bg: '#f9dfe3', text: '#0f172b' },
    ],
  },
  {
    label: 'Dry Goods',
    cells: [
      { value: 15, bg: '#e47081', text: '#ffffff' },
      { value: 60, bg: '#af2136', text: '#ffffff' },
      { value: 25, bg: '#e47081', text: '#ffffff' },
      { value: 10, bg: '#f9dfe3', text: '#0f172b' },
    ],
  },
  {
    label: 'Snacks',
    cells: [
      { value: 5, bg: '#f9dfe3', text: '#0f172b' },
      { value: 10, bg: '#f9dfe3', text: '#0f172b' },
      { value: 45, bg: '#da3850', text: '#ffffff' },
      { value: 20, bg: '#e47081', text: '#ffffff' },
    ],
  },
  {
    label: 'Beverages',
    cells: [
      { value: 3, bg: '#f9dfe3', text: '#0f172b' },
      { value: 8, bg: '#f9dfe3', text: '#0f172b' },
      { value: 12, bg: '#f9dfe3', text: '#0f172b' },
      { value: 20, bg: '#da3850', text: '#ffffff' },
    ],
  },
]

// Demand vs Supply — forecasted demand against actual supply received.
const demandSupplyByMonth = [
  { month: 'Mar', demand: 420, supply: 400 },
  { month: 'Apr', demand: 460, supply: 440 },
  { month: 'May', demand: 510, supply: 470 },
  { month: 'Jun', demand: 480, supply: 500 },
  { month: 'Jul', demand: 540, supply: 520 },
  { month: 'Aug', demand: 560, supply: 545 },
]
const demandSupplyChartOptions = {
  chart: { type: 'line', toolbar: { show: false }, animations: { enabled: false } },
  colors: ['#af2136', '#51a2ff'],
  stroke: { curve: 'smooth', width: 2 },
  markers: { size: 0 },
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: demandSupplyByMonth.map((point) => point.month),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false, min: 380, max: 600 },
  tooltip: {
    shared: true,
    custom: ({ series, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[0][dataPointIndex]} demand / ${series[1][dataPointIndex]} supply</div>
        <div class="chart-tooltip-label">${w.globals.labels[dataPointIndex]}</div>
      </div>
    `,
  },
}
const demandSupplySeries = [
  { name: 'Demand', data: demandSupplyByMonth.map((point) => point.demand) },
  { name: 'Supply', data: demandSupplyByMonth.map((point) => point.supply) },
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
                <img class="delta-icon" :src="stat.trend === 'up' ? iconTrendUp : iconMinus" alt="" />
                <span class="delta-value" :class="{ flat: stat.trend === 'flat' }">{{ stat.delta }}</span>
                <span class="delta-caption">vs last month</span>
              </div>
            </div>
            <div class="stat-icon-wrap" :style="{ background: stat.bg }">
              <img :src="stat.icon" :style="{ width: stat.iconW + 'px', height: stat.iconH + 'px' }" alt="" />
            </div>
          </div>
        </div>

        <!-- Inventory Forecasting + Stock Level Trend -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>Inventory Forecasting</h2>
              <p>Stockout count against 3-month forecast.</p>
            </div>
            <div class="line-chart-container">
              <VueApexCharts
                type="area"
                width="100%"
                height="100%"
                :options="inventoryForecastingChartOptions"
                :series="inventoryForecastingSeries"
              />
            </div>
            <div class="widget-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #2b7fff"></span><span>Actual</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #ff8904"></span><span>Forecast</span></div>
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
              <h2>Stock Level Trend</h2>
              <p>Quantity on hand across the last 30 days.</p>
            </div>
            <div class="line-chart-container">
              <VueApexCharts
                type="area"
                width="100%"
                height="100%"
                :options="stockLevelTrendChartOptions"
                :series="stockLevelTrendSeries"
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

        <!-- Inventory Turnover Ratio + ABC Analysis -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>Inventory Turnover Ratio</h2>
              <p>Turnover rate by category this month.</p>
            </div>
            <div class="h-bar-chart-container">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="turnoverChartOptions"
                :series="turnoverSeries"
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
              <h2>ABC Analysis</h2>
              <p>Items ranked by value contribution, classified into tiers.</p>
            </div>
            <div class="table-header-row">
              <span>Value (RM)</span>
              <span>Cumulative %</span>
            </div>
            <div class="combo-chart-container">
              <VueApexCharts
                type="line"
                width="100%"
                height="100%"
                :options="abcAnalysisChartOptions"
                :series="abcAnalysisSeries"
              />
            </div>
            <div class="widget-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #1447e6"></span><span>Tier A</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #2b7fff"></span><span>Tier B</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #8ec5ff"></span><span>Tier C</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #911b2d"></span><span>Cumulative %</span></div>
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

        <!-- Stockout Frequency + Supplier Lead Time -->
        <div class="widget-row">
          <div class="widget-card">
            <div class="card-header">
              <h2>Stockout Frequency</h2>
              <p>Number of stockout events by category.</p>
            </div>
            <div class="h-bar-chart-container">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="stockoutChartOptions"
                :series="stockoutSeries"
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
              <h2>Supplier Lead Time</h2>
              <p>Actual versus promised delivery time by supplier.</p>
            </div>
            <div class="table-header-row">
              <span>Supplier</span>
              <span>Variance (Days)</span>
            </div>
            <div class="dumbbell-chart">
              <div v-for="supplier in supplierLeadTimes" :key="supplier.name" class="dumbbell-row">
                <span class="dumbbell-label">{{ supplier.name }}</span>
                <div class="dumbbell-track">
                  <span
                    class="dumbbell-connector"
                    :style="{
                      left: Math.min(leadTimePercent(supplier.promised), leadTimePercent(supplier.actual)) + '%',
                      width: Math.abs(leadTimePercent(supplier.actual) - leadTimePercent(supplier.promised)) + '%',
                    }"
                  ></span>
                  <span class="dumbbell-value promised" :style="{ left: leadTimePercent(supplier.promised) + '%' }">{{ supplier.promised }}</span>
                  <span class="dumbbell-dot promised" :style="{ left: leadTimePercent(supplier.promised) + '%' }"></span>
                  <span class="dumbbell-dot actual" :style="{ left: leadTimePercent(supplier.actual) + '%' }"></span>
                  <span class="dumbbell-value actual" :style="{ left: leadTimePercent(supplier.actual) + '%' }">{{ supplier.actual }}</span>
                </div>
                <span class="dumbbell-delta" :class="supplier.trend">{{ supplier.delta }}</span>
              </div>
            </div>
            <div class="widget-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #99a1af"></span><span>Promised</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #af2136"></span><span>Actual</span></div>
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

        <!-- Inventory Value by Category + Slow-Moving/Dead Stock -->
        <div class="widget-row">
          <div class="widget-card donut-card">
            <div class="card-header">
              <h2>Inventory Value by Category</h2>
              <p>Distribution of inventory value across categories.</p>
            </div>
            <div class="donut-chart">
              <VueApexCharts
                type="donut"
                width="100%"
                height="100%"
                :options="inventoryValueChartOptions"
                :series="inventoryValueSeries"
              />
              <div class="donut-label">
                <p class="donut-percent">{{ inventoryTotalValueLabel }}</p>
                <p class="donut-caption">Total Value</p>
              </div>
            </div>
            <div class="widget-legend">
              <div v-for="category in inventoryValueByCategory" :key="category.label" class="legend-item">
                <span class="legend-dot" :style="{ background: category.color }"></span>
                <span>{{ category.label }}</span>
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
              <h2>Slow-Moving/ Dead Stock</h2>
              <p>Items with low or no movement over the last 90 days.</p>
            </div>
            <div class="heatmap-container">
              <div class="heatmap-rows">
                <div v-for="row in slowMovingHeatmapRows" :key="row.label" class="heatmap-row">
                  <span class="heatmap-row-label">{{ row.label }}</span>
                  <div class="heatmap-cells">
                    <span
                      v-for="(cell, i) in row.cells"
                      :key="i"
                      class="heatmap-cell"
                      :style="{ background: cell.bg, color: cell.text }"
                    >{{ cell.value }}</span>
                  </div>
                </div>
              </div>
              <div class="heatmap-columns">
                <span class="heatmap-columns-spacer"></span>
                <div class="heatmap-columns-labels">
                  <span v-for="wh in slowMovingWarehouses" :key="wh">{{ wh }}</span>
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
        </div>

        <!-- Demand vs Supply (full width) -->
        <div class="widget-row">
          <div class="widget-card full-width">
            <div class="card-header">
              <h2>Demand vs Supply</h2>
              <p>Forecasted demand against actual supply received.</p>
            </div>
            <div class="line-chart-container">
              <VueApexCharts
                type="line"
                width="100%"
                height="100%"
                :options="demandSupplyChartOptions"
                :series="demandSupplySeries"
              />
            </div>
            <div class="widget-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #af2136"></span><span>Demand</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #51a2ff"></span><span>Supply</span></div>
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
  --brand: #911b2d;
  --heading: #101828;
  --body-text: #4a5565;
  --border: #e5e7eb;
  --border-light: #f3f4f6;
  --success: #007a55;
  --warning: #d03801;
  --body-text-subtle: #6a7282;

  display: flex;
  align-items: stretch;
  width: 100%;
  min-height: 100vh;
  background: #ffffff;
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
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0) 28.397%,
    rgba(255, 255, 255, 0.6) 35.859%,
    rgba(255, 255, 255, 0.946) 45.87%,
    rgb(255, 255, 255) 100%
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
    linear-gradient(180deg, #ffd282 0%, #ba4255 100%) border-box;
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
.delta-value.flat {
  color: var(--body-text);
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
.widget-card.full-width {
  width: 100%;
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

/* Line / area charts (forecasting, stock level, demand vs supply) */
.line-chart-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}

/* Horizontal bar charts (turnover ratio, stockout frequency) */
.h-bar-chart-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: center;
  min-height: 0;
}

/* Combo bar+line chart (ABC analysis) */
.combo-chart-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: center;
  min-height: 0;
}

.table-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-shrink: 0;
  font-size: 13px;
  font-weight: 500;
  color: var(--body-text);
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

/* Donut card (inventory value by category) */
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

/* Dumbbell / range chart (supplier lead time) */
.dumbbell-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}
.dumbbell-row {
  display: flex;
  align-items: center;
  gap: 12px;
}
.dumbbell-label {
  flex-shrink: 0;
  width: 90px;
  font-size: 13px;
  font-weight: 500;
  color: var(--body-text);
}
.dumbbell-track {
  position: relative;
  flex: 1;
  height: 10px;
  background: #e2e8f0;
  border-radius: 9999px;
}
.dumbbell-connector {
  position: absolute;
  top: 0;
  height: 10px;
  background: #f9dfe3;
  border-radius: 9999px;
}
.dumbbell-dot {
  position: absolute;
  top: 50%;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}
.dumbbell-dot.promised {
  background: #99a1af;
  z-index: 1;
}
.dumbbell-dot.actual {
  background: #af2136;
  z-index: 2;
}
.dumbbell-value {
  position: absolute;
  top: -20px;
  transform: translateX(-50%);
  font-size: 12px;
  font-weight: 600;
  color: var(--heading);
  white-space: nowrap;
}
.dumbbell-delta {
  flex-shrink: 0;
  width: 48px;
  font-size: 13px;
  font-weight: 700;
  text-align: right;
}
.dumbbell-delta.over {
  color: var(--warning);
}
.dumbbell-delta.under {
  color: var(--success);
}

/* Heatmap grid (slow-moving/dead stock) */
.heatmap-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 8px;
  min-height: 0;
}
.heatmap-rows {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 4px;
  min-height: 0;
}
.heatmap-row {
  display: flex;
  align-items: stretch;
  flex: 1;
  gap: 12px;
  min-height: 0;
}
.heatmap-row-label {
  flex-shrink: 0;
  width: 72px;
  display: flex;
  align-items: center;
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
}
.heatmap-cells {
  display: flex;
  flex: 1;
  gap: 2px;
  min-width: 0;
}
.heatmap-cell {
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
}
.heatmap-columns {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-shrink: 0;
}
.heatmap-columns-spacer {
  flex-shrink: 0;
  width: 72px;
}
.heatmap-columns-labels {
  display: flex;
  flex: 1;
  gap: 2px;
}
.heatmap-columns-labels span {
  flex: 1;
  text-align: center;
  font-size: 14px;
  font-weight: 500;
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
