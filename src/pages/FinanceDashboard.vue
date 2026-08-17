<script setup>
/**
 * FinanceDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1115-23370
 *
 * Finance module landing dashboard: sidebar + topnav shell around expense/profit
 * KPI cards and spending charts. Reached from the workspace selector's "Finance"
 * module card.
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
import iconBarChart from '../assets/dashboard/sidebar/licensing/icon-bar-chart.svg'
import iconStoreAlt from '../assets/dashboard/sidebar/licensing/icon-store-alt.svg'
import iconFileLines from '../assets/dashboard/sidebar/licensing/icon-file-lines.svg'
import iconArrowsRepeat from '../assets/dashboard/sidebar/licensing/icon-arrows-repeat.svg'
import iconBuilding from '../assets/dashboard/sidebar/finance/icon-building.svg'
import iconClockArrow from '../assets/dashboard/sidebar/finance/icon-clock-arrow.svg'
import iconUsers from '../assets/dashboard/sidebar/finance/icon-users.svg'
import iconCheckCircle from '../assets/dashboard/sidebar/finance/icon-check-circle.svg'
import iconTicket from '../assets/dashboard/sidebar/finance/icon-ticket.svg'
import iconCash from '../assets/dashboard/sidebar/finance/icon-cash.svg'
import iconDatabase from '../assets/dashboard/sidebar/finance/icon-database.svg'
import iconFileInvoice from '../assets/dashboard/sidebar/finance/icon-file-invoice.svg'

import imgPatternBg from '../assets/dashboard/pattern-bg.jpg'

const emit = defineEmits(['back-to-workspace', 'sign-out'])

// Sidebar module list for the Finance workspace. None of these open
// sub-navigation in the Figma design (no chevron affordance), unlike the
// Licensing module's sidebar.
const navItems = [
  { key: 'overview', icon: iconGrid, label: 'Overview', width: 15, height: 15 },
  { key: 'item-category', icon: iconTag, label: 'Item Category', width: 15, height: 15 },
  { key: 'company', icon: iconBuilding, label: 'Company', width: 13.33, height: 15 },
  { key: 'order-history', icon: iconClockArrow, label: 'Order History', width: 16.67, height: 16.67 },
  { key: 'order-request', icon: iconCart, label: 'Order Request', width: 13.33, height: 15 },
  { key: 'outlet-sales-analysis', icon: iconBarChart, label: 'Outlet Sales Analysis', width: 16.67, height: 15 },
  { key: 'outlet', icon: iconStoreAlt, label: 'Outlet', width: 16.67, height: 16.67 },
  { key: 'vendor-registration-review', icon: iconUsers, label: 'Vendor Registration Review', width: 13.33, height: 16.67 },
  { key: 'pr-approvals', icon: iconCheckCircle, label: 'PR Approvals', width: 16.67, height: 16.67 },
  { key: 'purchase-order-deliveries', icon: iconFileLines, label: 'Purchase Order Deliveries', width: 13.33, height: 16.67 },
  { key: 'trade-in', icon: iconArrowsRepeat, label: 'Trade In', width: 13.33, height: 15 },
  { key: 'redemption-history', icon: iconTicket, label: 'Redemption History', width: 16.67, height: 11.67 },
  { key: 'licensee-fees', icon: iconCash, label: 'Licensee Fees', width: 16.67, height: 13.33 },
  { key: 'sql-data', icon: iconDatabase, label: 'SQL Data', width: 13.33, height: 16.67 },
  { key: 'invoice-entities', icon: iconFileInvoice, label: 'Invoice Entities', width: 13.33, height: 16.67 },
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
  { index: '1', category: 'Invoice Approvals', count: '(12 Items)' },
  { index: '2', category: 'Purchase Order Deliveries', count: '(7 Items)' },
  { index: '3', category: 'Vendor Registration Review', count: '(5 Items)' },
  { index: '4', category: 'PR Approvals', count: '(3 Items)' },
  { index: '5', category: 'Trade In Applications', count: '(9 Items)' },
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
  { key: 'total-expenses', label: 'Total Expenses', value: 'MYR 4,150,567', delta: '0%', bg: '#fef2f2', icon: iconStatTotalExpenses, iconW: 15, iconH: 15 },
  { key: 'other-expenses', label: 'Other Expenses', value: 'MYR 39,403', delta: '0%', bg: '#fffbeb', icon: iconStatOtherExpenses, iconW: 11.67, iconH: 16.67 },
  { key: 'net-profit', label: 'Net Profit', value: '15,000', delta: '0%', bg: '#f0fdf4', icon: iconStatNetProfit, iconW: 15, iconH: 14.58 },
]

function formatMyr(value) {
  return `MYR ${Math.round(value).toLocaleString('en-MY')}`
}

const salesVsExpenses = [
  { day: 'Mon', sales: 58000, expenses: 50000 },
  { day: 'Tue', sales: 64000, expenses: 56000 },
  { day: 'Wed', sales: 52000, expenses: 56000 },
  { day: 'Thu', sales: 38000, expenses: 42000 },
  { day: 'Fri', sales: 54000, expenses: 46000 },
  { day: 'Sat', sales: 58000, expenses: 56000 },
  { day: 'Sun', sales: 64000, expenses: 56000 },
]

const salesVsExpensesChartOptions = {
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
  colors: ['#8EC5FF', '#1447E6'],
  dataLabels: { enabled: false },
  legend: { show: false },
  grid: { show: false },
  xaxis: {
    categories: salesVsExpenses.map((point) => point.day),
    labels: { style: { colors: '#4a5565', fontSize: '14px', fontWeight: 500 } },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: { show: false },
  tooltip: {
    custom: ({ series, seriesIndex, dataPointIndex, w }) => `
      <div class="chart-tooltip">
        <div class="chart-tooltip-value">${formatMyr(series[seriesIndex][dataPointIndex])}</div>
        <div class="chart-tooltip-label">${w.globals.seriesNames[seriesIndex]} · ${w.globals.categoryLabels[dataPointIndex]}</div>
      </div>
    `,
  },
}

const salesVsExpensesSeries = [
  { name: 'Sales', data: salesVsExpenses.map((point) => point.sales) },
  { name: 'Expenses', data: salesVsExpenses.map((point) => point.expenses) },
]

const expenseBreakdown = [
  { label: 'Office', value: 38.4, color: '#05df72' },
  { label: 'Travel', value: 28.2, color: '#00b8db' },
  { label: 'Marketing', value: 17.8, color: '#fdc700' },
  { label: 'Utilities', value: 8.2, color: '#ff6467' },
  { label: 'Salaries', value: 7.4, color: '#c27aff' },
]

const donutChartOptions = {
  chart: {
    type: 'donut',
    toolbar: { show: false },
    animations: { enabled: false },
  },
  labels: expenseBreakdown.map((category) => category.label),
  colors: expenseBreakdown.map((category) => category.color),
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

const donutSeries = expenseBreakdown.map((category) => category.value)
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
              <h2>Sales vs. Expenses</h2>
              <p>Daily comparison of sales and expenses across the week.</p>
            </div>
            <div class="bar-chart">
              <VueApexCharts
                type="bar"
                width="100%"
                height="100%"
                :options="salesVsExpensesChartOptions"
                :series="salesVsExpensesSeries"
              />
            </div>
            <div class="bar-chart-legend">
              <div class="legend-item"><span class="legend-dot" style="background: #8ec5ff"></span><span>Sales</span></div>
              <div class="legend-item"><span class="legend-dot" style="background: #1447e6"></span><span>Expenses</span></div>
            </div>
            <div class="card-footer">
              <span>Last Updated : Today, 2:45PM</span>
              <a href="#" class="see-details">
                See Details
                <img :src="iconChevronDetails" alt="" />
              </a>
            </div>
          </div>

          <div class="donut-card">
            <div class="card-header">
              <h2>Expense Breakdown By Category</h2>
              <p>Distribution of expenses across spending categories.</p>
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
                <p class="donut-percent">{{ expenseBreakdown[0].value }}%</p>
                <p class="donut-caption">{{ expenseBreakdown[0].label }}</p>
              </div>
            </div>
            <div class="donut-legend">
              <div v-for="category in expenseBreakdown" :key="category.label" class="legend-item">
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

/* Widget grid (two-up row). auto-fit/minmax reflows off the grid's own
   rendered width, so it collapses to one column whenever the layers panel
   (or a narrow viewport) leaves too little room for two. */
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

/* Bar chart (sales vs. expenses) */
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

/* Donut card (expense breakdown) */
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
.donut-card .card-header,
.donut-card .card-footer {
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
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
}

/* ---------- Responsive ---------- */
@media (max-width: 1100px) {
  .chart-card,
  .donut-card {
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
  .dashboard.layers-panel-open .dashboard-main {
    margin-right: 0;
  }
  .layers-panel {
    top: 0;
    width: 100%;
  }
}
</style>
