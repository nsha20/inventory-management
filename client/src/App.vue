<template>
  <div class="app-shell">
    <Sidebar
      @show-profile-details="showProfileDetails = true"
      @show-tasks="showTasks = true"
    />

    <div class="app-body">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

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
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import Sidebar from './components/Sidebar.vue'
import FilterBar from './components/FilterBar.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'

export default {
  name: 'App',
  components: {
    Sidebar,
    FilterBar,
    ProfileDetailsModal,
    TasksModal
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

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

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
/* ── Design Tokens ───────────────────────────────────────────────────── */
:root {
  /* Sidebar */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 60px;
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-active-bg: rgba(37, 99, 235, 0.15);
  --sidebar-active-text: #93c5fd;
  --sidebar-active-icon: #60a5fa;
  --sidebar-hover-bg: rgba(255, 255, 255, 0.06);
  --sidebar-border: rgba(255, 255, 255, 0.08);
  --sidebar-border-subtle: rgba(255, 255, 255, 0.08);

  /* Page */
  --color-bg: #f1f5f9;
  --color-surface: #ffffff;
  --color-border: #e2e8f0;
  --color-text: #1e293b;
  --color-muted: #64748b;
  --color-accent: #2563eb;

  /* Spacing scale (4px base) */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-5: 1.25rem;   /* 20px */
  --space-6: 1.5rem;    /* 24px */
  --space-7: 1.75rem;   /* 28px */
  --space-8: 2rem;      /* 32px */

  /* Typography */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
}

/* ── Reset ───────────────────────────────────────────────────────────── */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── Shell: sidebar + scrollable body ───────────────────────────────── */
.app-shell {
  display: flex;
  height: 100vh;
  overflow: hidden;
  background: var(--color-bg);
}

.app-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-width: 0;
}

/* ── Main content area ───────────────────────────────────────────────── */
.main-content {
  flex: 1;
  overflow-y: auto;
  padding: var(--space-6) var(--space-7);
}

/* ── Page-level typography ───────────────────────────────────────────── */
.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: 1.75rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--color-muted);
  font-size: var(--text-sm);
}

/* ── Stat cards ──────────────────────────────────────────────────────── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5);
  border-radius: 10px;
  border: 1px solid var(--color-border);
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: var(--color-muted);
  font-size: 0.813rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value { color: #ea580c; }
.stat-card.success .stat-value { color: #059669; }
.stat-card.danger .stat-value  { color: #dc2626; }
.stat-card.info .stat-value    { color: var(--color-accent); }

/* ── Generic card ────────────────────────────────────────────────────── */
.card {
  background: var(--color-surface);
  border-radius: 10px;
  padding: var(--space-5);
  border: 1px solid var(--color-border);
  margin-bottom: var(--space-5);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: 1.063rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.015em;
}

/* ── Table ───────────────────────────────────────────────────────────── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid var(--color-border);
  border-bottom: 1px solid var(--color-border);
}

th {
  text-align: left;
  padding: var(--space-2) var(--space-3);
  font-weight: 600;
  color: #475569;
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: var(--space-2) var(--space-3);
  border-top: 1px solid var(--color-bg);
  color: #334155;
  font-size: var(--text-sm);
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

/* ── Badges ──────────────────────────────────────────────────────────── */
.badge {
  display: inline-block;
  padding: var(--space-1) 0.625rem;
  border-radius: 6px;
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fed7aa; color: #92400e; }
.badge.danger     { background: #fecaca; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fecaca; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fecaca; color: #991b1b; }
.badge.medium     { background: #fed7aa; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* ── State ───────────────────────────────────────────────────────────── */
.loading {
  text-align: center;
  padding: var(--space-8);
  color: var(--color-muted);
  font-size: var(--text-sm);
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: 8px;
  margin: var(--space-4) 0;
  font-size: var(--text-sm);
}
</style>
