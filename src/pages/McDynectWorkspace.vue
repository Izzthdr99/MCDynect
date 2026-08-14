<script setup>
/**
 * McDynectWorkspace.vue
 * Figma source: https://www.figma.com/design/6WiB15odMAP0hE7ufDP3iA/Mc-Dynect-26?node-id=1281-4420
 *
 * Post-login workspace/module selector. Shown after a successful login.
 * Icons and the pattern background are exported from Figma and committed locally
 * (see src/assets/workspace/) rather than pointed at Figma's temporary asset host.
 */
import iconSuperAdmin from '../assets/workspace/icon-super-admin.svg'
import iconLicensing from '../assets/workspace/icon-licensing.svg'
import iconLicensee from '../assets/workspace/icon-licensee.svg'
import iconOperationManager from '../assets/workspace/icon-operation-manager.svg'
import iconAreaManager from '../assets/workspace/icon-area-manager.svg'
import iconFinance from '../assets/workspace/icon-finance.svg'
import iconProcurement from '../assets/workspace/icon-procurement.svg'
import iconProduction from '../assets/workspace/icon-production.svg'
import iconInventory from '../assets/workspace/icon-inventory.svg'
import iconCompliance from '../assets/workspace/icon-compliance.svg'
import iconChevron from '../assets/workspace/icon-chevron.svg'
import iconChevronActive from '../assets/workspace/icon-chevron-active.svg'
import imgPatternBackground from '../assets/workspace/pattern-bg.jpg'

const emit = defineEmits(['select'])

const modules = [
  { key: 'super-admin', icon: iconSuperAdmin, title: 'Super Admin', description: 'Manage tenants, users, and permissions on the platform.' },
  { key: 'licensing', icon: iconLicensing, title: 'Licensing', description: 'Issue licenses, track renewals, and manage accounts.' },
  { key: 'licensee', icon: iconLicensee, title: 'Licensee', description: 'Check your license status, renewal dates, and entitlements.' },
  { key: 'operation-manager', icon: iconOperationManager, title: 'Outlet Manager', description: 'Manage daily operations, handle escalations, and track KPIs.' },
  { key: 'area-manager', icon: iconAreaManager, title: 'Area Manager', description: 'Oversee local outlets — staffing, performance, and operations.' },
  { key: 'finance', icon: iconFinance, title: 'Finance', description: 'Handle invoicing, payments, and financial reports.' },
  { key: 'procurement', icon: iconProcurement, title: 'Procurement', description: 'Submit requisitions, manage vendors, and process orders.' },
  { key: 'production', icon: iconProduction, title: 'Production', description: 'Plan production runs and monitor output targets.' },
  { key: 'inventory', icon: iconInventory, title: 'Inventory', description: 'Manage stock, warehouses, and deliveries.' },
  { key: 'compliance', icon: iconCompliance, title: 'Compliance', description: 'Oversee audits and compliance across the business.' },
]

function handleSelect(moduleKey) {
  emit('select', moduleKey)
}
</script>

<template>
  <div class="page">
    <div class="pattern-bg" :style="{ backgroundImage: 'url(' + imgPatternBackground + ')' }"></div>
    <div class="fade-edge fade-left"></div>
    <div class="fade-edge fade-right"></div>

    <div class="content">
      <h1 class="heading"><span class="heading-accent">Access and manage</span> your business modules from a single workspace.</h1>

      <div class="modules-grid">
        <button
          v-for="mod in modules"
          :key="mod.key"
          type="button"
          class="module-card"
          @click="handleSelect(mod.key)"
        >
          <div class="module-card-header">
            <div class="module-card-title">
              <img class="module-icon" :src="mod.icon" alt="" />
              <span>{{ mod.title }}</span>
            </div>
            <span class="link-button" aria-hidden="true">
              <img class="chevron chevron-rest" :src="iconChevron" alt="" />
              <img class="chevron chevron-active" :src="iconChevronActive" alt="" />
            </span>
          </div>
          <p class="module-card-description">{{ mod.description }}</p>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.page {
  --brand: #b13444;
  --brand-strong: #54101a;
  --accent: #d53f52;
  --hover-border: #ffd282;
  --text-body: #4a5565;

  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  width: 100%;
  padding: 80px 40px;
  background: #fffdf8;
  overflow: hidden;
}

.page * {
  box-sizing: border-box;
}

.pattern-bg {
  position: absolute;
  inset: 0;
  background-size: 900px auto;
  background-repeat: repeat;
  opacity: 0.5;
  pointer-events: none;
}

.fade-edge {
  position: absolute;
  top: 0;
  width: 220px;
  height: 100%;
  pointer-events: none;
}
.fade-left {
  left: 0;
  background: linear-gradient(to left, rgba(255, 255, 255, 0), #fff);
}
.fade-right {
  right: 0;
  background: linear-gradient(to right, rgba(255, 255, 255, 0), #fff);
}

.content {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 56px;
  width: 100%;
  max-width: 896px;
}

.heading {
  margin: 0;
  width: 100%;
  font-size: 36px;
  font-weight: 600;
  line-height: normal;
  color: var(--brand-strong);
  text-align: left;
}
.heading-accent {
  color: var(--accent);
}

.modules-grid {
  display: grid;
  grid-template-columns: repeat(3, 288px);
  gap: 16px;
  justify-content: center;
  width: 100%;
}

.module-card {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 16px;
  width: 288px;
  min-height: 140px;
  padding: 28px 20px;
  background: rgba(255, 255, 255, 0.5);
  border: 1px solid #f4f2ef;
  border-radius: 16px;
  box-shadow: 0px 4px 12px 0px rgba(175, 33, 54, 0.05);
  font-family: inherit;
  text-align: left;
  cursor: pointer;
  transition: background 0.15s ease, border-color 0.15s ease, box-shadow 0.15s ease;
}
.module-card:hover,
.module-card:focus-visible {
  background: #ffffff;
  border-color: var(--hover-border);
  box-shadow: 0px 4px 6px 0px rgba(175, 33, 54, 0.05);
  outline: none;
}

.module-card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  width: 100%;
}

.module-card-title {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 0;
}
.module-icon {
  width: 24px;
  height: 24px;
  object-fit: contain;
  flex-shrink: 0;
}
.module-card-title span {
  font-family: 'Gotham Rounded', inherit;
  font-weight: 500;
  font-size: 18px;
  color: var(--accent);
  white-space: nowrap;
}

.link-button {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  background: transparent;
  flex-shrink: 0;
  transition: background 0.15s ease;
}
.module-card:hover .link-button,
.module-card:focus-visible .link-button {
  background: var(--brand);
}

.chevron {
  position: absolute;
  width: 16px;
  height: 9px;
  transform: rotate(90deg);
  transition: opacity 0.15s ease;
}
.chevron-rest {
  opacity: 1;
}
.chevron-active {
  opacity: 0;
}
.module-card:hover .chevron-rest,
.module-card:focus-visible .chevron-rest {
  opacity: 0;
}
.module-card:hover .chevron-active,
.module-card:focus-visible .chevron-active {
  opacity: 1;
}

.module-card-description {
  margin: 0;
  font-size: 14px;
  line-height: 1.5;
  color: var(--text-body);
}

/* ---------- Responsive ---------- */
@media (max-width: 980px) {
  .modules-grid {
    grid-template-columns: repeat(2, 288px);
  }
}
@media (max-width: 660px) {
  .page {
    padding: 48px 20px;
  }
  .modules-grid {
    grid-template-columns: 1fr;
    max-width: 320px;
  }
  .module-card {
    width: 100%;
  }
  .heading {
    font-size: 28px;
  }
}
</style>
