<template>
  <aside
    id="app-sidebar"
    class="sidebar"
    :class="{ collapsed, 'mobile-open': mobileOpen }"
  >
    <div class="sidebar-header">
      <div class="logo">
        <span class="logo-mark" aria-hidden="true">CC</span>
        <span class="logo-text" v-show="!collapsed">
          <span class="logo-name">{{ t('nav.companyName') }}</span>
          <span class="logo-sub">{{ t('nav.subtitle') }}</span>
        </span>
      </div>
    </div>

    <nav class="sidebar-nav" aria-label="Primary">
      <span class="nav-section-label" v-show="!collapsed">Main</span>
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :class="{ active: $route.path === item.to }"
        :aria-current="$route.path === item.to ? 'page' : null"
        @click="$emit('close-mobile')"
      >
        <span class="nav-icon" v-html="item.icon" aria-hidden="true" />
        <span class="nav-label" v-show="!collapsed">{{ item.label }}</span>
      </router-link>
    </nav>

    <button
      class="sidebar-collapse-btn"
      type="button"
      @click="$emit('toggle-collapse')"
      :aria-label="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
      :aria-expanded="!collapsed"
    >
      <svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
        <polyline :points="collapsed ? '9 18 15 12 9 6' : '15 18 9 12 15 6'"/>
      </svg>
    </button>
  </aside>

  <div
    class="sidebar-backdrop"
    v-show="mobileOpen"
    @click="$emit('close-mobile')"
    aria-hidden="true"
  />
</template>

<script>
import { computed } from 'vue'
import { useI18n } from '../composables/useI18n'

const ICONS = {
  overview:   '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="9"/><rect x="14" y="3" width="7" height="5"/><rect x="14" y="12" width="7" height="9"/><rect x="3" y="16" width="7" height="5"/></svg>',
  inventory:  '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>',
  orders:     '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 11H3v9h6"/><path d="M21 11h-6v9h6"/><path d="M3 11V4h18v7"/></svg>',
  finance:    '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>',
  demand:     '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/></svg>',
  reports:    '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>',
}

export default {
  name: 'AppSidebar',
  props: {
    collapsed:  { type: Boolean, default: false },
    mobileOpen: { type: Boolean, default: false },
  },
  emits: ['toggle-collapse', 'close-mobile'],
  setup() {
    const { t } = useI18n()
    const navItems = computed(() => ([
      { to: '/',          label: t('nav.overview'),       icon: ICONS.overview  },
      { to: '/inventory', label: t('nav.inventory'),      icon: ICONS.inventory },
      { to: '/orders',    label: t('nav.orders'),         icon: ICONS.orders    },
      { to: '/spending',  label: t('nav.finance'),        icon: ICONS.finance   },
      { to: '/demand',    label: t('nav.demandForecast'), icon: ICONS.demand    },
      { to: '/reports',   label: 'Reports',               icon: ICONS.reports   },
    ]))
    return { t, navItems }
  },
}
</script>

<style scoped>
.sidebar {
  background: var(--bg-surface);
  border-right: 1px solid var(--border-subtle);
  display: flex;
  flex-direction: column;
  padding: var(--space-4) var(--space-3);
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  transition: width var(--transition-base);
}
.sidebar-header { padding: var(--space-2) var(--space-2) var(--space-4); }
.logo { display: flex; align-items: center; gap: var(--space-3); }
.logo-mark {
  width: 32px; height: 32px; flex-shrink: 0;
  background: var(--color-accent);
  color: white;
  border-radius: var(--radius-sm);
  display: inline-flex; align-items: center; justify-content: center;
  font-weight: 700; font-size: 13px; letter-spacing: -0.02em;
}
.logo-text { display: flex; flex-direction: column; line-height: 1.2; }
.logo-name { font-weight: 700; color: var(--text-primary); font-size: 0.9375rem; }
.logo-sub  { font-size: 0.75rem; color: var(--text-secondary); }

.sidebar-nav { display: flex; flex-direction: column; gap: var(--space-1); flex: 1; }
.nav-section-label {
  font-size: 0.6875rem; font-weight: 600; text-transform: uppercase;
  letter-spacing: 0.08em; color: var(--text-muted);
  padding: var(--space-3) var(--space-3) var(--space-2);
}
.nav-item {
  display: flex; align-items: center; gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-sm);
  color: var(--text-secondary);
  text-decoration: none;
  font-size: 0.9375rem; font-weight: 500;
  transition: background var(--transition-fast), color var(--transition-fast);
}
.nav-item:hover { background: var(--color-slate-100); color: var(--text-primary); }
.nav-item.active {
  background: var(--color-accent-soft);
  color: var(--color-accent);
  font-weight: 600;
}
.nav-icon { display: inline-flex; flex-shrink: 0; }
.nav-label { white-space: nowrap; }
.nav-item:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

.sidebar-collapse-btn {
  margin-top: var(--space-4);
  align-self: flex-end;
  width: 32px; height: 32px;
  background: transparent;
  border: 1px solid var(--border-subtle);
  border-radius: var(--radius-sm);
  color: var(--text-secondary);
  cursor: pointer;
  display: inline-flex; align-items: center; justify-content: center;
}
.sidebar-collapse-btn:hover { background: var(--color-slate-100); color: var(--text-primary); }
.sidebar-collapse-btn:focus-visible {
  outline: 2px solid var(--color-accent); outline-offset: 2px;
}

.sidebar.collapsed .sidebar-collapse-btn { align-self: center; }

.sidebar-backdrop { display: none; }

@media (max-width: 768px) {
  .sidebar {
    position: fixed; inset: 0 auto 0 0;
    width: 280px; height: 100vh;
    transform: translateX(-100%);
    transition: transform var(--transition-base);
    z-index: 200;
  }
  .sidebar.mobile-open { transform: translateX(0); box-shadow: var(--shadow-lg); }
  .sidebar-backdrop {
    display: block;
    position: fixed; inset: 0;
    background: rgba(15,23,42,0.4);
    z-index: 150;
  }
}
</style>
