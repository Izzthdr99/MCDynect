<script setup>
/**
 * LicensingDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1027-22257
 *
 * Licensing module landing dashboard: sidebar + topnav shell around licensee
 * KPI cards, revenue trend, outlet coverage map, and restock widgets.
 * Reached from the workspace selector's "Licensing" module card.
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
import iconChevronSort from '../assets/dashboard/topnav/icon-chevron-sort.svg'
import iconStatActiveLicensee from '../assets/dashboard/icons/icon-stat-active-licensee.svg'
import iconStatGroupSales from '../assets/dashboard/icons/icon-stat-group-sales.svg'
import iconStatComboRestock from '../assets/dashboard/icons/icon-stat-combo-restock.svg'
import imgWorldMap from '../assets/dashboard/licensing/world-map.svg'
import imgPatternBg from '../assets/dashboard/pattern-bg.jpg'

import iconGrid from '../assets/dashboard/sidebar/licensing/icon-grid.svg'
import iconFileLines from '../assets/dashboard/sidebar/licensing/icon-file-lines.svg'
import iconCart from '../assets/dashboard/sidebar/licensing/icon-cart.svg'
import iconClipboardList from '../assets/dashboard/sidebar/licensing/icon-clipboard-list.svg'
import iconTag from '../assets/dashboard/sidebar/licensing/icon-tag.svg'
import iconStoreAlt from '../assets/dashboard/sidebar/licensing/icon-store-alt.svg'
import iconMapPinAlt from '../assets/dashboard/sidebar/licensing/icon-map-pin-alt.svg'
import iconBadgeCheck from '../assets/dashboard/sidebar/licensing/icon-badge-check.svg'
import iconRefresh from '../assets/dashboard/sidebar/licensing/icon-refresh.svg'
import iconGiftBox from '../assets/dashboard/sidebar/licensing/icon-gift-box.svg'
import iconArrowsRepeat from '../assets/dashboard/sidebar/licensing/icon-arrows-repeat.svg'
import iconBarChart from '../assets/dashboard/sidebar/licensing/icon-bar-chart.svg'
import iconSettings from '../assets/dashboard/sidebar/icon-settings.svg'

const emit = defineEmits(['back-to-workspace', 'sign-out'])

const activeNavItem = ref('overview')
const sidebarCollapsed = ref(false)

// Sidebar module list swapped in for the Licensing workspace. Items with a
// trailing chevron (Outlet Management, Licensing End to End, etc.) open
// sub-navigation that isn't built yet, matching the Figma design.
const navItems = [
  { key: 'overview', icon: iconGrid, label: 'Overview', width: 15, height: 15 },
  { key: 'manage-memo', icon: iconFileLines, label: 'Manage Memo', width: 13.33, height: 16.67 },
  { key: 'order-request', icon: iconCart, label: 'Order Request', width: 13.33, height: 15 },
  { key: 'form-application', icon: iconClipboardList, label: 'Form Application', width: 13.33, height: 16.67 },
  { key: 'event-price-settings', icon: iconTag, label: 'Event Price Settings', width: 15, height: 15 },
  { key: 'flexi-operation', icon: iconSettings, label: 'Flexi Operation', width: 16.67, height: 16.67 },
  { key: 'bazaar-application', icon: iconStoreAlt, label: 'Bazaar Application', width: 16.67, height: 16.67 },
  { key: 'outlet-management', icon: iconMapPinAlt, label: 'Outlet Management', width: 13.33, height: 16.67, chevron: true },
  { key: 'licensing-end-to-end', icon: iconBadgeCheck, label: 'Licensing End to End', width: 16.67, height: 16.67, chevron: true },
  { key: 'program-renewals', icon: iconRefresh, label: 'Program Renewals', width: 13.38, height: 15, chevron: true },
  { key: 'starter-pack-promotions', icon: iconGiftBox, label: 'Starter Pack & Promotions', width: 16.67, height: 16.67, chevron: true },
  { key: 'trade-in', icon: iconArrowsRepeat, label: 'Trade In', width: 13.33, height: 15, chevron: true },
  { key: 'analysis', icon: iconBarChart, label: 'Analysis', width: 16.67, height: 15, chevron: true },
]

function handleNavigate(key) {
  // Only Overview is built; sub-navigation items are no-ops for now.
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
  { index: '1', category: 'Program Renewals', count: '(12 Items)' },
  { index: '2', category: 'Licensing End to End', count: '(7 Items)' },
  { index: '3', category: 'Trade In Applications', count: '(5 Items)' },
  { index: '4', category: 'Starter Pack & Promotions', count: '(3 Items)' },
  { index: '5', category: 'Bazaar Applications', count: '(9 Items)' },
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
  { key: 'active-licensee', label: 'Active Licensee', value: '150', delta: '0%', bg: '#faf5ff', icon: iconStatActiveLicensee, iconW: 16.67, iconH: 16.67 },
  { key: 'group-total-sales', label: 'Group Total Sales', value: 'MYR 39,403', delta: '0%', bg: '#f0fdf4', icon: iconStatGroupSales, iconW: 15, iconH: 14.58 },
  { key: 'total-combo-restock', label: 'Total Combo Restock', value: '15,000', delta: '0%', bg: '#fffbeb', icon: iconStatComboRestock, iconW: 13.84, iconH: 15 },
]

function formatMyr(value) {
  return `MYR ${Math.round(value).toLocaleString('en-MY')}`
}

const licenseeTrend = [
  { day: 'Jan 21', value: 15200 },
  { day: 'Jan 22', value: 21400 },
  { day: 'Jan 23', value: 28900 },
  { day: 'Jan 25', value: 17800 },
  { day: 'Jan 26', value: 24600 },
  { day: 'Jan 27', value: 19300 },
]

const licenseeTrendChartOptions = {
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
    categories: licenseeTrend.map((point) => point.day),
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

const licenseeTrendSeries = [{ name: 'Revenue', data: licenseeTrend.map((point) => point.value) }]

const outletPerformance = [
  { name: 'Mr Churros Bandar Sri Uda', amount: 'RM 60,750' },
  { name: 'Mr Churros Bandar Sri Uda II', amount: 'RM 52,000' },
  { name: 'Mr Churros Taman Melati', amount: 'RM 47,200' },
  { name: 'Mr Churros Kota Damansara', amount: 'RM 45,000' },
  { name: 'Mr Churros Bukit Bintang', amount: 'RM 45,000' },
  { name: 'Mr Churros Bandar Banting', amount: 'RM 38,500' },
]

const comboRestockByState = [
  { key: 'combo-set', name: '[Package] Combo Set', percent: 41 },
  { key: 'dipping-container', name: 'Dipping Container', percent: 22 },
  { key: 'combo-equipment', name: 'Combo Equipment', percent: 15 },
  { key: 'cinnamon-sugar', name: 'Cinnamon Sugar', percent: 12 },
  { key: 'collectable-sticker', name: 'Collectable Sticker', percent: 8 },
  { key: 'cotton-candy', name: 'Cotton Candy', percent: 2 },
]

const stateRestockVsSales = [
  { week: 'Week 1', restock: 630, sales: 760 },
  { week: 'Week 2', restock: 630, sales: 840 },
  { week: 'Week 3', restock: 180, sales: 1000 },
  { week: 'Week 4', restock: 750, sales: 300 },
]

const stateRestockChartOptions = {
  chart: {
    type: 'bar',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  plotOptions: {
    bar: {
      columnWidth: '55%',
      borderRadius: 4,
    },
  },
  colors: ['#1447E6', '#8EC5FF'],
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: stateRestockVsSales.map((point) => point.week),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${series[seriesIndex][dataPointIndex]} units</div>
        <div class="chart-tooltip-label">${w.globals.seriesNames[seriesIndex]} · ${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const stateRestockSeries = [
  { name: 'Restock', data: stateRestockVsSales.map((point) => point.restock) },
  { name: 'Sales', data: stateRestockVsSales.map((point) => point.sales) },
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
              <h2>Licensee Trend (Daily)</h2>
              <p>Daily revenue trend across all outlets.</p>
            </div>
            <div class="line-chart">
              <div class="line-chart-canvas">
                <VueApexCharts
                  type="area"
                  width="100%"
                  height="100%"
                  :options="licenseeTrendChartOptions"
                  :series="licenseeTrendSeries"
                />
              </div>
              <div class="chart-x-axis">
                <span v-for="point in licenseeTrend" :key="point.day">{{ point.day }}</span>
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

          <div class="map-card">
            <div class="card-header">
              <h2>Outlet Count by State</h2>
              <p>Number of active outlets per state, Malaysia.</p>
            </div>
            <div class="world-map">
              <img :src="imgWorldMap" alt="World map highlighting states with active outlets" />
            </div>
          </div>
        </div>

        <div class="widget-grid">
          <div class="table-card">
            <div class="card-header table-card-header">
              <h2>Outlet Performance Table</h2>
              <p>Top-performing outlets ranked by revenue this period.</p>
            </div>
            <div class="data-table">
              <div class="data-table-header">
                <span class="data-table-col-name">Page</span>
                <span class="data-table-col-amount">
                  Revenu
                  <img class="sort-icon" :src="iconChevronSort" alt="" />
                </span>
              </div>
              <div v-for="outlet in outletPerformance" :key="outlet.name" class="data-table-row">
                <span class="data-table-col-name">{{ outlet.name }}</span>
                <span class="data-table-col-amount">{{ outlet.amount }}</span>
              </div>
            </div>
          </div>

          <div class="chart-card">
            <div class="card-header">
              <h2>Total Combo Restock By State</h2>
              <p>Restock volume share by combo item, this period.</p>
            </div>
            <div class="progress-list">
              <div v-for="item in comboRestockByState" :key="item.key" class="progress-row">
                <span class="progress-label">{{ item.name }}</span>
                <div class="progress-track">
                  <div class="progress-fill" :style="{ width: item.percent + '%' }"></div>
                </div>
                <span class="progress-value">{{ item.percent }}%</span>
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

        <div class="widget-grid">
          <div class="chart-card">
            <div class="card-header">
              <h2>Licensee Trend (Daily)</h2>
              <p>Daily revenue trend across all outlets.</p>
            </div>
            <div class="line-chart">
              <div class="line-chart-canvas">
                <VueApexCharts
                  type="area"
                  width="100%"
                  height="100%"
                  :options="licenseeTrendChartOptions"
                  :series="licenseeTrendSeries"
                />
              </div>
              <div class="chart-x-axis">
                <span v-for="point in licenseeTrend" :key="point.day">{{ point.day }}</span>
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
              <h2>State Restock vs. Sales</h2>
              <p>Restock volume compared to sales revenue, by week.</p>
            </div>
            <div class="bar-chart">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="stateRestockChartOptions"
                :series="stateRestockSeries"
              />
            </div>
            <div class="bar-chart-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #1447e6"></span><span>Restock</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #8ec5ff"></span><span>Sales</span></div>
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

/* Widget grid (two-up rows). auto-fit/minmax reflows off the grid's own
   rendered width, so it collapses to one column whenever the layers panel
   (or a narrow viewport) leaves too little room for two. */
.widget-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 24px;
}

.chart-card,
.map-card,
.table-card {
  display: flex;
  flex-direction: column;
  height: 462px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 16px;
  min-width: 0;
}
.chart-card,
.map-card {
  gap: 24px;
  padding: 20px;
}
/* The map graphic is a ~3MB SVG (350+ paths, 176 shadow filters) — without
   containment, opening/closing the layers panel forces the browser to
   repaint it on every frame of the sibling `.dashboard-main` margin-right
   transition, even though its own box never actually changes size. `contain`
   isolates its layout/paint from that ancestor change, and `will-change`
   promotes it to its own compositor layer so the browser can skip re-doing
   that work each frame. */
.map-card {
  content-visibility: auto;
  contain-intrinsic-size: 462px 462px;
  contain: strict;
  will-change: transform;
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

/* World map card */
.world-map {
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
  min-height: 0;
}
.world-map img {
  width: 100%;
  height: auto;
  max-height: 100%;
  object-fit: contain;
}

/* Outlet Performance table */
.table-card-header {
  padding: 20px 20px 16px;
  border-bottom: none;
}
.data-table {
  display: flex;
  flex: 1;
  flex-direction: column;
  min-height: 0;
  overflow-y: auto;
}
.data-table-header,
.data-table-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 0 20px;
  flex-shrink: 0;
}
.data-table-header {
  height: 44px;
  background: #f9fafb;
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
}
.data-table-row {
  height: 56px;
  border-bottom: 1px solid var(--border-light);
}
.data-table-row:last-child {
  border-bottom: none;
}
.data-table-col-name {
  flex: 1;
  min-width: 0;
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.data-table-header .data-table-col-name,
.data-table-header .data-table-col-amount {
  font-size: 14px;
  font-weight: 500;
  color: var(--body-text);
}
.data-table-col-amount {
  display: flex;
  align-items: center;
  gap: 4px;
  flex-shrink: 0;
  font-size: 14px;
  font-weight: 500;
  color: var(--heading);
}
.sort-icon {
  width: 6.67px;
  height: 10.67px;
}

/* Combo restock progress list */
.progress-list {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}
.progress-row {
  display: flex;
  align-items: center;
  gap: 32px;
}
.progress-label {
  flex-shrink: 0;
  width: 144px;
  font-size: 16px;
  font-weight: 500;
  color: var(--heading);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.progress-track {
  position: relative;
  flex: 1;
  min-width: 0;
  height: 10px;
  background: #f3f4f6;
  border-radius: 9999px;
  overflow: hidden;
}
.progress-fill {
  height: 100%;
  background: #818cf8;
  border-radius: 9999px;
}
.progress-value {
  flex-shrink: 0;
  width: 40px;
  font-size: 16px;
  font-weight: 500;
  color: var(--heading);
  text-align: right;
}

/* Vertical bar chart (state restock vs sales) */
.bar-chart {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-evenly;
  min-height: 0;
}
.bar-chart-legend {
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

/* ---------- Responsive ---------- */
@media (max-width: 1100px) {
  .chart-card,
  .map-card,
  .table-card {
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
