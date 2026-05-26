---
name: saas-redesign
description: Redesigns the Vue 3 frontend into a modern SaaS-style UI with a vertical left sidebar, design tokens, and consistent spacing. Use this skill when the user asks to modernize the UI, add a sidebar layout, introduce a design system, or apply a SaaS-style visual refresh to the client application.
---

# SaaS-Style UI Redesign

This skill turns the current top-nav Vue 3 frontend into a modern SaaS dashboard: a fixed left sidebar with collapsible + responsive behavior, a slimmed top bar, a token-based design system, and consistent spacing across all global classes.

The skill is opinionated for **this** project — it names real files, preserves the Catalyst Components branding, the slate palette, and every existing composable. It does not touch routing, views, modals, or backend code.

## When to invoke this skill

Invoke when the user asks for any of:

- "Modernize the UI", "SaaS-style redesign", "polish the design"
- "Move the navigation to a left sidebar"
- "Add a design system / design tokens / CSS variables"
- "Make spacing consistent"
- "Make the layout responsive"

Do **not** invoke for one-off color tweaks, single-component restyling, or routing changes.

## Critical Rule — Delegate every `.vue` edit to `vue-expert`

The project's `CLAUDE.md` mandates: **ANY creation or significant modification of a `.vue` file MUST go through the `vue-expert` agent** (via the Task tool). This skill is the recipe; `vue-expert` writes the code.

Your job when this skill is invoked:

1. Read this skill and the relevant existing files (`App.vue`, `FilterBar.vue`).
2. For each `.vue` change, delegate to `vue-expert` with a precise prompt that quotes the relevant section of this skill.
3. After all `.vue` edits are done, run the verification plan yourself.

Do **not** edit `.vue` files directly — even small ones. The single non-`.vue` edit (none currently planned) would be fine.

## Scope

### Files this skill touches

| File | Action |
|---|---|
| `client/src/App.vue` | Modify — new grid shell, tokens at top of global `<style>`, slimmed top bar, global class refactor, ESC handler, two local refs |
| `client/src/components/AppSidebar.vue` | **Create** — new component |
| `client/src/components/FilterBar.vue` | Modify — single CSS-only change (sticky `top` offset) |

### Files this skill must leave alone

- `client/src/components/LanguageSwitcher.vue`
- `client/src/components/ProfileMenu.vue`
- `client/src/components/*Modal.vue` (all of them)
- `client/src/views/*.vue` (Dashboard, Inventory, Orders, Demand, Spending, Reports, Backlog)
- `client/src/composables/{useAuth,useI18n,useFilters}.js`
- `client/src/main.js` (router)
- `client/src/locales/{en,ja}.js` (sidebar reuses existing `nav.*` keys)
- `client/src/api.js`
- Anything under `server/` or `tests/`

## Step-by-step execution order

When invoked, follow these phases in order. Each `.vue` step is a separate `vue-expert` delegation.

1. **Read** `client/src/App.vue` and `client/src/components/FilterBar.vue` to confirm current state matches the assumptions below.
2. **Delegate to `vue-expert`** — insert design tokens at the top of the global `<style>` in `App.vue`. (Section: Design Tokens)
3. **Delegate to `vue-expert`** — restructure the `<template>` of `App.vue` into the new grid shell, add local `isCollapsed` / `isMobileOpen` refs, add ESC handler. (Section: App.vue Shell)
4. **Delegate to `vue-expert`** — create `client/src/components/AppSidebar.vue`. (Section: AppSidebar Component)
5. **Delegate to `vue-expert`** — refactor the global classes in `App.vue` against the new tokens; delete `.top-nav` and `.nav-tabs` blocks; add `.top-bar` and `.hamburger`. (Section: Global Class Refactor)
6. **Delegate to `vue-expert`** — update FilterBar's sticky `top` value. (Section: FilterBar Adjustment)
7. **Delegate to `vue-expert`** — add the responsive media query and accessibility styles. (Section: Responsive + Accessibility)
8. **Verify** with `./scripts/start.sh` + Playwright MCP. (Section: Verification)

Each delegation prompt should reference this skill by name and quote the relevant section, so `vue-expert` has unambiguous instructions.

## Design Tokens

Insert this block at the top of `App.vue`'s global `<style>` (immediately after the `*` reset). Every other style rule should be rewritten to reference these tokens instead of hardcoded hex.

```css
:root {
  /* Slate scale */
  --color-slate-50:  #f8fafc;
  --color-slate-100: #f1f5f9;
  --color-slate-200: #e2e8f0;
  --color-slate-300: #cbd5e1;
  --color-slate-400: #94a3b8;
  --color-slate-500: #64748b;
  --color-slate-600: #475569;
  --color-slate-700: #334155;
  --color-slate-800: #1e293b;
  --color-slate-900: #0f172a;

  /* Brand + semantic */
  --color-accent:       #2563eb;
  --color-accent-soft:  #eff6ff;
  --color-success:      #059669;
  --color-success-soft: #d1fae5;
  --color-warning:      #ea580c;
  --color-warning-soft: #fed7aa;
  --color-danger:       #dc2626;
  --color-danger-soft:  #fecaca;
  --color-info-soft:    #dbeafe;

  /* Surfaces */
  --bg-app:        var(--color-slate-50);
  --bg-surface:    #ffffff;
  --border-subtle: var(--color-slate-200);
  --border-strong: var(--color-slate-300);
  --text-primary:   var(--color-slate-900);
  --text-secondary: var(--color-slate-500);
  --text-muted:     var(--color-slate-400);

  /* Spacing scale */
  --space-1:  0.25rem;
  --space-2:  0.5rem;
  --space-3:  0.75rem;
  --space-4:  1rem;
  --space-5:  1.25rem;
  --space-6:  1.5rem;
  --space-8:  2rem;
  --space-12: 3rem;

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;

  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgba(15,23,42,.04);
  --shadow-md: 0 4px 12px rgba(15,23,42,.06);
  --shadow-lg: 0 10px 24px rgba(15,23,42,.08);

  /* Motion */
  --transition-fast: 0.15s ease;
  --transition-base: 0.20s ease;

  /* Layout */
  --sidebar-w:           240px;
  --sidebar-w-collapsed: 64px;
  --topbar-h:            56px;
  --content-max-w:       1600px;
}
```

Rules:
- Do not change the actual color values — these are the existing palette, just named.
- Spacing scale skips `--space-7`, `--space-9`, etc. by design (4/8/12/16/20/24/32/48 progression).
- No dark-mode override block in scope — tokens enable it later, but do not add `[data-theme="dark"]` rules.

## App.vue Shell

### New grid layout

```css
.app {
  display: grid;
  grid-template-columns: var(--sidebar-w) 1fr;
  grid-template-rows: var(--topbar-h) auto 1fr;
  grid-template-areas:
    "sidebar topbar"
    "sidebar filterbar"
    "sidebar main";
  min-height: 100vh;
  transition: grid-template-columns var(--transition-base);
}
.app.sidebar-collapsed { --sidebar-w: var(--sidebar-w-collapsed); }
.sidebar     { grid-area: sidebar; }
.top-bar     { grid-area: topbar; }
.filters-bar { grid-area: filterbar; }
.main-content{ grid-area: main; }
```

### New template

```vue
<template>
  <div
    class="app"
    :class="{
      'sidebar-collapsed': isCollapsed,
      'sidebar-mobile-open': isMobileOpen
    }"
  >
    <AppSidebar
      :collapsed="isCollapsed"
      :mobile-open="isMobileOpen"
      @toggle-collapse="isCollapsed = !isCollapsed"
      @close-mobile="isMobileOpen = false"
    />

    <header class="top-bar">
      <button
        class="hamburger"
        type="button"
        @click="isMobileOpen = true"
        aria-label="Open navigation menu"
        aria-controls="app-sidebar"
      >
        <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <line x1="3" y1="6" x2="21" y2="6"/>
          <line x1="3" y1="12" x2="21" y2="12"/>
          <line x1="3" y1="18" x2="21" y2="18"/>
        </svg>
      </button>
      <div class="top-bar-spacer" />
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="showProfileDetails = true"
        @show-tasks="showTasks = true"
      />
    </header>

    <FilterBar />

    <main class="main-content">
      <router-view />
    </main>

    <!-- Existing modals — leave untouched -->
    <ProfileDetailsModal :is-open="showProfileDetails" @close="showProfileDetails = false" />
    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>
```

### Script additions

Inside `setup()`, add (alongside the existing refs):

```js
import { ref, onMounted, onUnmounted, computed } from 'vue'
import AppSidebar from './components/AppSidebar.vue'   // new import

// ...existing useAuth, useI18n, refs, tasks logic...

const isCollapsed   = ref(false)
const isMobileOpen  = ref(false)

const onKeydown = (e) => {
  if (e.key === 'Escape') isMobileOpen.value = false
}

onMounted(() => {
  loadTasks()                                  // existing call
  window.addEventListener('keydown', onKeydown)
})
onUnmounted(() => {
  window.removeEventListener('keydown', onKeydown)
})

return {
  // ...existing returns...
  isCollapsed,
  isMobileOpen,
}
```

Also register `AppSidebar` in the `components: { ... }` object.

### State location rationale

`isCollapsed` and `isMobileOpen` live as **local refs in `App.vue`**, not a new `useSidebar` composable. Only two components consume the state (App + AppSidebar), which is below the threshold set by `client/CLAUDE.md` for introducing composables ("state shared across multiple components"). If localStorage persistence or keyboard shortcuts are added later, refactor to `useSidebar` at that point.

## AppSidebar Component

Create `client/src/components/AppSidebar.vue`. Skeleton:

```vue
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
```

Tips for `vue-expert`:
- Keep the SVGs inline (no icon library).
- Use the existing `t('nav.*')` i18n keys; `nav.reports` is not in locales — fall back to the hardcoded "Reports" string (matches `App.vue` line 26).
- `aria-current` should be `null` (not `'false'`) on inactive links so the attribute is omitted entirely.

## Top Bar Slim-down

The new `.top-bar` replaces `.top-nav`:

```css
.top-bar {
  background: var(--bg-surface);
  border-bottom: 1px solid var(--border-subtle);
  box-shadow: var(--shadow-sm);
  height: var(--topbar-h);
  padding: 0 var(--space-6);
  display: flex; align-items: center; gap: var(--space-3);
  position: sticky; top: 0; z-index: 100;
}
.top-bar-spacer { flex: 1; }

.hamburger {
  display: none; /* shown only on mobile via @media */
  width: 36px; height: 36px;
  background: transparent;
  border: 1px solid var(--border-subtle);
  border-radius: var(--radius-sm);
  color: var(--text-primary);
  cursor: pointer;
  align-items: center; justify-content: center;
}
.hamburger:hover { background: var(--color-slate-100); }
.hamburger:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 2px; }
```

Delete the entire `.top-nav`, `.nav-container`, `.nav-container > .nav-tabs`, `.nav-container > .language-switcher`, `.logo`, `.logo h1`, `.subtitle`, `.nav-tabs`, `.nav-tabs a`, `.nav-tabs a:hover`, `.nav-tabs a.active`, and `.nav-tabs a.active::after` blocks from `App.vue`. The logo is now inside the sidebar; the nav links live in the sidebar.

## FilterBar Adjustment

Single change in `client/src/components/FilterBar.vue` scoped `<style>`:

```css
/* Before */
.filters-bar {
  /* ... */
  position: sticky;
  top: 70px;   /* matched old top-nav height */
}

/* After */
.filters-bar {
  /* ... */
  position: sticky;
  top: var(--topbar-h);   /* sticks under new slimmed top bar */
}
```

Do **not** change any of: the `<template>`, `<script>`, the `useFilters` import, the select options, the i18n keys, or any other CSS rule. This is a single-line edit.

## Global Class Refactor (Spacing Audit)

Rewrite the following blocks in `App.vue`'s global `<style>` to use tokens. Values shown reflect the existing computed values, just expressed through tokens — visual output should be identical except for the spacing tweaks called out.

```css
body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--bg-app);
  color: var(--color-slate-800);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.main-content {
  max-width: var(--content-max-w);
  width: 100%;
  margin: 0 auto;
  padding: var(--space-6) var(--space-8);
}

.page-header { margin-bottom: var(--space-6); }
.page-header h2 {
  font-size: 1.875rem; font-weight: 700;
  color: var(--text-primary);
  margin-bottom: var(--space-1);
  letter-spacing: -0.025em;
}
.page-header p { color: var(--text-secondary); font-size: 0.9375rem; }

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--bg-surface);
  padding: var(--space-5);
  border-radius: var(--radius-md);
  border: 1px solid var(--border-subtle);
  transition: border-color var(--transition-base), box-shadow var(--transition-base);
}
.stat-card:hover {
  border-color: var(--border-strong);
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--text-secondary);
  font-size: 0.875rem; font-weight: 600;
  text-transform: uppercase; letter-spacing: 0.5px;
  margin-bottom: var(--space-2);
}
.stat-value {
  font-size: 2.25rem; font-weight: 700;
  color: var(--text-primary); letter-spacing: -0.025em;
}
.stat-card.warning .stat-value { color: var(--color-warning); }
.stat-card.success .stat-value { color: var(--color-success); }
.stat-card.danger  .stat-value { color: var(--color-danger);  }
.stat-card.info    .stat-value { color: var(--color-accent);  }

.card {
  background: var(--bg-surface);
  border-radius: var(--radius-md);
  padding: var(--space-5);
  border: 1px solid var(--border-subtle);
  margin-bottom: var(--space-5);
}
.card-header {
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 1px solid var(--border-subtle);
}
.card-title {
  font-size: 1.125rem; font-weight: 700;
  color: var(--text-primary); letter-spacing: -0.025em;
}

.table-container { overflow-x: auto; }
table { width: 100%; border-collapse: collapse; }
thead {
  background: var(--color-slate-50);
  border-top: 1px solid var(--border-subtle);
  border-bottom: 1px solid var(--border-subtle);
}
th {
  text-align: left; padding: var(--space-2) var(--space-3);
  font-weight: 600; color: var(--color-slate-600);
  font-size: 0.75rem; text-transform: uppercase; letter-spacing: 0.05em;
}
td {
  padding: var(--space-2) var(--space-3);
  border-top: 1px solid var(--color-slate-100);
  color: var(--color-slate-700);
  font-size: 0.875rem;
}
tbody tr { transition: background-color var(--transition-fast); }
tbody tr:hover { background: var(--color-slate-50); }

.badge {
  display: inline-block;
  padding: 0.313rem var(--space-3);
  border-radius: var(--radius-sm);
  font-size: 0.75rem; font-weight: 600;
  text-transform: uppercase; letter-spacing: 0.025em;
}
.badge.success, .badge.increasing { background: var(--color-success-soft); color: #065f46; }
.badge.warning, .badge.medium     { background: var(--color-warning-soft); color: #92400e; }
.badge.danger, .badge.decreasing, .badge.high { background: var(--color-danger-soft); color: #991b1b; }
.badge.info, .badge.low           { background: var(--color-info-soft);    color: #1e40af; }
.badge.stable                      { background: #e0e7ff;                   color: #3730a3; }

.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--text-secondary);
  font-size: 0.9375rem;
}
.error {
  background: #fef2f2;
  border: 1px solid var(--color-danger-soft);
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-sm);
  margin: var(--space-4) 0;
  font-size: 0.9375rem;
}
```

Also add a global focus ring for keyboard nav:

```css
:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 2px; border-radius: var(--radius-sm); }
```

## Responsive Behavior

Add at the bottom of `App.vue`'s global `<style>`:

```css
@media (max-width: 768px) {
  .app {
    grid-template-columns: 1fr;
    grid-template-areas:
      "topbar"
      "filterbar"
      "main";
  }
  .hamburger { display: inline-flex; }
  .main-content { padding: var(--space-4) var(--space-4); }
}
```

The sidebar's own scoped style already handles its mobile drawer transform (see AppSidebar component).

## Accessibility Checklist

The `vue-expert` delegations must ensure all of these:

- `<nav aria-label="Primary">` wraps the sidebar nav list.
- Each active link gets `aria-current="page"` (null when inactive — omit the attribute).
- `:focus-visible` outline on nav links, hamburger, and collapse button.
- Collapse button: `aria-label` toggles between "Collapse sidebar" / "Expand sidebar"; `aria-expanded` reflects state.
- Hamburger button: `aria-label="Open navigation menu"`, `aria-controls="app-sidebar"`.
- The sidebar `<aside>` carries `id="app-sidebar"` so `aria-controls` resolves.
- ESC key closes the mobile drawer (handler in `App.vue` `setup()`).
- Clicking the backdrop closes the mobile drawer.
- Clicking any nav link on mobile closes the drawer (`@click="$emit('close-mobile')"`).

## Verification

After all delegations are complete:

1. Boot the stack:
   ```bash
   ./scripts/start.sh
   ```
   Frontend at http://localhost:3000, backend at http://localhost:8001.

2. Use Playwright MCP (`mcp__playwright__*`) to:
   - Navigate to `http://localhost:3000/`. Take a screenshot at viewport 1440×900. Confirm sidebar on left, slimmed top bar at top, FilterBar sticky beneath it.
   - Click each sidebar item: `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports`. Confirm exactly one `.nav-item.active` at a time and the URL updates.
   - Click the collapse toggle. Take a screenshot at the same viewport. Confirm labels are hidden, icons remain, content area widens, no horizontal scrollbar.
   - Resize viewport to 375×812. Confirm sidebar is hidden, hamburger visible in the top bar.
   - Click the hamburger. Take a screenshot. Confirm drawer slides in with backdrop.
   - Press ESC. Confirm drawer closes.
   - Change a `FilterBar` select (e.g. Time Period → 2025-06). Confirm dashboard data updates — this is the sanity check that `useFilters` wiring is intact.

3. Backend tests are UI-agnostic — they should still pass without changes:
   ```bash
   cd tests && uv run pytest -v
   ```

If any of the above fails, fix via additional `vue-expert` delegations referencing this skill.

## Non-Goals

This skill explicitly does NOT:

- Modify the router (`client/src/main.js`).
- Modify composables (`useAuth`, `useI18n`, `useFilters`).
- Redesign any view (`Dashboard.vue`, `Inventory.vue`, etc.) — they benefit only via global-class restyling.
- Add dark mode (tokens enable it later, but no `[data-theme="dark"]` block here).
- Modify any modal component.
- Add new dependencies (no Tailwind, no icon library, no UI kit).
- Touch the backend, API, or tests.
- Migrate FilterBar into the sidebar.
- Remove or change the "Catalyst Components" branding.
- Persist `isCollapsed` to localStorage (out-of-scope follow-up).

## Common Gotchas

- **Do** delegate every `.vue` change to `vue-expert`. Do not edit `.vue` files yourself.
- **Do** check the diff of `App.vue` carefully — the global style block is long and easy to leave dead `.nav-tabs` rules in.
- **Do** verify `FilterBar.vue`'s only change is the `top` value; if `vue-expert` rewrites more, ask it to revert the unrelated edits.
- **Don't** rename existing CSS classes used by views (`.card`, `.stat-card`, `.badge`, etc.) — views depend on them. Refactor the rules in place.
- **Don't** introduce a `useSidebar` composable yet — local refs are sufficient. Defer until persistence is needed.
- **Don't** add a 6th sidebar item or remove an existing one without user direction. Keep the six existing routes.
- **Don't** add an `nav.reports` i18n key — the current code uses the hardcoded "Reports" string at `App.vue` line 26; preserve that to keep the diff minimal.
- **Don't** change the 1600px content max-width to a smaller value; users are familiar with it.

## Out-of-scope follow-ups

Mention these to the user only if they ask "what's next" — do not execute as part of this skill:

- Extract `useSidebar` composable with localStorage persistence of `isCollapsed`.
- Add a `[data-theme="dark"]` token override block for dark mode.
- Migrate the "Reports" label to i18n (`nav.reports` in `en.js` / `ja.js`).
- Swap inline SVGs for a `lucide-vue-next` icon library once nav items grow past ~8.
- Add a search input or command palette to the sidebar.
