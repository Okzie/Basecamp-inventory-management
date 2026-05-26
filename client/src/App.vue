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

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

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

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import AppSidebar from './components/AppSidebar.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher,
    AppSidebar
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const isCollapsed  = ref(false)
    const isMobileOpen = ref(false)

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    const onKeydown = (e) => {
      if (e.key === 'Escape') isMobileOpen.value = false
    }

    onMounted(() => {
      loadTasks()
      window.addEventListener('keydown', onKeydown)
    })
    onUnmounted(() => {
      window.removeEventListener('keydown', onKeydown)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      isCollapsed,
      isMobileOpen,
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

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

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--bg-app);
  color: var(--color-slate-800);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 2px; border-radius: var(--radius-sm); }

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
  display: none;
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
</style>
