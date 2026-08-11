<script setup>
/**
 * SuperAdminDashboard.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=926-12145
 *
 * SuperAdmin module landing dashboard: sidebar + topnav shell around sales
 * KPI cards, trend charts, and revenue ranking tables. Reached from the
 * workspace selector's "Super Admin" module card.
 */
import { ref } from 'vue'
import DashboardSidebar from '../components/DashboardSidebar.vue'
import DashboardTopnav from '../components/DashboardTopnav.vue'

import iconAngleDown from '../assets/dashboard/icons/icon-angle-down.svg'
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
import chartSalesTrend from '../assets/dashboard/charts/chart-sales-trend.svg'
import chartCumulativeSales from '../assets/dashboard/charts/chart-cumulative-sales.svg'
import chartDonutChannel from '../assets/dashboard/charts/chart-donut-channel.svg'

const emit = defineEmits(['back-to-workspace'])

const activeNavItem = ref('dashboard')
const sidebarCollapsed = ref(false)

function handleNavigate(key) {
  // Only Dashboard is built; other routes are no-ops for now.
  if (key === 'dashboard') activeNavItem.value = key
}

// Icon width/height match each SVG's own exported Figma bounding box —
// forcing them all into one square (as an earlier pass did) stretches
// every icon that isn't already square.
const statCards = [
  { key: 'today-sales', label: "Today's Sales", value: 'MYR 24,580', delta: '0%', bg: '#eef6ff', icon: iconStatSales, iconW: 11.67, iconH: 16.67 },
  { key: 'mtd-sales', label: 'MTD Sales', value: 'MYR 612,900', delta: '0%', bg: '#f0fdf4', icon: iconStatMtd, iconW: 15, iconH: 15 },
  { key: 'total-kg', label: 'Total KG Sold', value: '18,240 kg', delta: '0%', bg: '#eef2ff', icon: iconStatKg, iconW: 13.33, iconH: 16.67 },
  { key: 'unsold-waste', label: 'Unsold / Waste', value: '342 sets', delta: '10%', bg: '#fef2f2', icon: iconStatWaste, iconW: 13.33, iconH: 16.67 },
  { key: 'avg-sales-kg', label: 'Avg. Sales / KG', value: 'MYR 33.60', delta: '5%', bg: '#faf5ff', icon: iconStatAvg, iconW: 16.67, iconH: 13.33 },
]

const salesTrendDays = ['Jan 21', 'Jan 22', 'Jan 23', 'Jan 25', 'Jan 26', 'Jan 27']
const cumulativeSalesDays = ['Jan 21', 'Jan 22', 'Jan 23', 'Jan 24', 'Jan 25', 'Jan 26', 'Jan 27']

const salesByOutlet = [
  { rank: 1, name: 'Mr Churros Bandar Sri Uda', location: 'Kuala Lumpur', amount: 'RM 34,000', trophy: iconTrophyGold },
  { rank: 2, name: 'Mr Churros Bandar Sri Uda II', location: 'Kuala Lumpur', amount: 'RM 30,000', trophy: iconTrophySilver },
  { rank: 3, name: 'Mr Churros Taman Melati', location: 'Kuala Lumpur', amount: 'RM 29,000', trophy: iconTrophyBronze },
  { rank: 4, name: 'Mr Churros Kota Damansara', location: 'Kuala Lumpur', amount: 'RM 14,000', trophy: null },
  { rank: 5, name: 'Mr Churros Bukit Bintang', location: 'Kuala Lumpur', amount: 'RM 10,000', trophy: null },
]

const salesByLicensee = [
  { key: 'licensee-1', widthPct: 43.92, color: '#0ba5ec' },
  { key: 'licensee-2', widthPct: 87.25, color: '#fd853a' },
  { key: 'licensee-3', widthPct: 64.9, color: '#0ba5ec' },
  { key: 'licensee-4', widthPct: 19.61, color: '#fd853a' },
  { key: 'licensee-5', widthPct: 36.47, color: '#0ba5ec' },
  { key: 'licensee-6', widthPct: 57.65, color: '#fd853a' },
]

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
  <div class="dashboard" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <DashboardSidebar :active-item="activeNavItem" :collapsed="sidebarCollapsed" @navigate="handleNavigate" />

    <div class="dashboard-main">
      <DashboardTopnav
        @toggle-sidebar="sidebarCollapsed = !sidebarCollapsed"
        @switch-workspace="emit('back-to-workspace')"
      />

      <div class="dashboard-content">
        <div class="header-section">
          <div class="heading-block">
            <h1 class="heading">Hello, Izzthdr.</h1>
            <p class="subheading">Here's what's happening today.</p>
          </div>
          <div class="header-actions">
            <button type="button" class="filter-button">
              <span>Filter</span>
              <img :src="iconAngleDown" alt="" />
            </button>
            <button type="button" class="layers-button" aria-label="View layers">
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
              <img :src="chartSalesTrend" alt="Daily sales trend chart" />
              <div class="chart-x-axis">
                <span v-for="day in salesTrendDays" :key="day">{{ day }}</span>
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
              <img :src="chartCumulativeSales" alt="Cumulative sales chart" />
              <div class="chart-x-axis">
                <span v-for="day in cumulativeSalesDays" :key="day">{{ day }}</span>
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
              <div
                v-for="bar in salesByLicensee"
                :key="bar.key"
                class="bar-track"
              >
                <div class="bar-fill" :style="{ width: bar.widthPct + '%', background: bar.color }"></div>
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
          <div class="donut-card">
            <div class="card-header">
              <h2>Sales By Channel</h2>
              <p>Dine-in, delivery, takeaway.</p>
            </div>
            <div class="donut-chart">
              <img :src="chartDonutChannel" alt="Sales by channel donut chart" />
              <div class="donut-label">
                <p class="donut-percent">82.4%</p>
                <p class="donut-caption">Emergency Fund</p>
              </div>
            </div>
            <div class="donut-legend">
              <div class="legend-item">
                <span class="legend-dot" style="background: #2b7fff"></span>
                <span>Cash</span>
              </div>
              <div class="legend-item">
                <span class="legend-dot" style="background: #d9d9d9"></span>
                <span>Cash Equivalent</span>
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

  display: flex;
  align-items: stretch;
  width: 100%;
  min-height: 100vh;
  background: #f9fafb;
  font-family: inherit;
}
.dashboard * {
  box-sizing: border-box;
}
.dashboard.sidebar-collapsed .dashboard-main {
  margin-left: 72px;
}

.dashboard-main {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
  /* Sidebar is position: fixed (out of flex flow), so its width has to be
     reserved here manually instead of the flexbox row sizing it. */
  margin-left: 251px;
  transition: margin-left 220ms ease;
}

.dashboard-content {
  display: flex;
  flex-direction: column;
  gap: 24px;
  padding: 32px;
  max-width: 1189px;
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
  width: 13.33px;
  height: 16.67px;
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
  width: 263px;
  flex-shrink: 0;
  padding: 24px;
  background: #ffffff;
  border: 1px solid #ffd282;
  border-radius: 12px;
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

/* Widget grid (two-up chart rows) */
.widget-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
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
.line-chart img {
  flex: 1;
  min-height: 0;
  width: 100%;
  object-fit: contain;
  transition: transform 180ms ease, filter 180ms ease;
}
.line-chart:hover img {
  transform: scale(1.02);
  filter: drop-shadow(0 4px 10px rgba(16, 24, 40, 0.12));
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
.bar-track {
  width: 100%;
  height: 26px;
  cursor: pointer;
}
.bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: filter 180ms ease, box-shadow 180ms ease;
}
.bar-track:hover .bar-fill {
  filter: brightness(1.08);
  box-shadow: 0 2px 8px rgba(16, 24, 40, 0.15);
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
.donut-chart img {
  width: 100%;
  height: 100%;
  transition: transform 180ms ease, filter 180ms ease;
}
.donut-chart:hover img {
  transform: scale(1.02);
  filter: drop-shadow(0 4px 10px rgba(16, 24, 40, 0.12));
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
  .widget-grid {
    grid-template-columns: 1fr;
  }
  .chart-card,
  .donut-card {
    height: auto;
  }
  .line-chart img {
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
}
</style>
