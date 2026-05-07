<template>
  <aside class="sidebar" :class="{ collapsed }">
    <!-- Brand -->
    <div class="sidebar-brand">
      <div class="brand-icon">
        <svg width="18" height="18" viewBox="0 0 22 22" fill="none">
          <rect x="2" y="2" width="8" height="8" rx="2" fill="currentColor" opacity="0.9"/>
          <rect x="12" y="2" width="8" height="8" rx="2" fill="currentColor" opacity="0.6"/>
          <rect x="2" y="12" width="8" height="8" rx="2" fill="currentColor" opacity="0.6"/>
          <rect x="12" y="12" width="8" height="8" rx="2" fill="currentColor" opacity="0.9"/>
        </svg>
      </div>
      <transition name="fade-label">
        <div v-if="!collapsed" class="brand-text">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-sub">{{ t('nav.subtitle') }}</span>
        </div>
      </transition>
      <button
        v-if="!collapsed"
        class="collapse-btn-brand"
        @click="collapsed = !collapsed"
        title="Collapse sidebar"
      >
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
        </svg>
      </button>
    </div>

    <!-- Nav -->
    <nav class="sidebar-nav">
      <div class="nav-section">
        <span v-if="!collapsed" class="nav-section-label">Navigation</span>
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="nav-link"
          :title="collapsed ? item.label : undefined"
        >
          <span class="nav-icon" v-html="item.icon" />
          <transition name="fade-label">
            <span v-if="!collapsed" class="nav-label">{{ item.label }}</span>
          </transition>
        </router-link>
      </div>
    </nav>

    <!-- Footer: Language + Profile -->
    <div class="sidebar-footer">
      <!-- Language -->
      <div class="footer-lang">
        <button
          class="footer-lang-btn"
          @click="toggleLangMenu"
          @blur="handleLangBlur"
          :title="collapsed ? 'Language' : undefined"
        >
          <svg width="18" height="18" viewBox="0 0 20 20" fill="none" class="footer-icon">
            <circle cx="10" cy="10" r="7.5" stroke="currentColor" stroke-width="1.5"/>
            <path d="M3 10H17" stroke="currentColor" stroke-width="1.5"/>
            <path d="M10 3C10 3 7.5 5.5 7.5 10C7.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
            <path d="M10 3C10 3 12.5 5.5 12.5 10C12.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          <transition name="fade-label">
            <span v-if="!collapsed" class="footer-btn-label">{{ localeName }}</span>
          </transition>
        </button>

        <div v-if="langMenuOpen" class="footer-dropdown lang-dropdown">
          <button
            v-for="locale in availableLocales"
            :key="locale"
            class="footer-dropdown-item"
            :class="{ active: currentLocale === locale }"
            @mousedown.prevent="selectLanguage(locale)"
          >
            <span>{{ getLanguageName(locale) }}</span>
            <svg v-if="currentLocale === locale" width="16" height="16" viewBox="0 0 18 18" fill="none">
              <path d="M4 9L7.5 12.5L14 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </button>
        </div>
      </div>

      <div class="footer-divider"></div>

      <!-- Profile -->
      <div class="footer-profile">
        <button
          class="footer-profile-btn"
          @click="toggleProfileMenu"
          @blur="handleProfileBlur"
          :title="collapsed ? currentUser.name : undefined"
        >
          <div class="avatar">{{ getInitials(currentUser.name) }}</div>
          <transition name="fade-label">
            <div v-if="!collapsed" class="profile-info">
              <span class="profile-name">{{ currentUser.name }}</span>
              <span class="profile-role">{{ currentUser.jobTitle }}</span>
            </div>
          </transition>
          <transition name="fade-label">
            <svg v-if="!collapsed" class="chevron" :class="{ 'chevron-up': profileMenuOpen }" width="16" height="16" viewBox="0 0 16 16" fill="none">
              <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/>
            </svg>
          </transition>
        </button>

        <div v-if="profileMenuOpen" class="footer-dropdown profile-dropdown">
          <div class="profile-dropdown-header">
            <div class="avatar-lg">{{ getInitials(currentUser.name) }}</div>
            <div class="profile-dropdown-info">
              <span class="profile-dropdown-name">{{ currentUser.name }}</span>
              <span class="profile-dropdown-email">{{ currentUser.email }}</span>
            </div>
          </div>
          <div class="footer-dropdown-divider"></div>
          <button class="footer-dropdown-item" @mousedown.prevent="openProfileDetails">
            <svg width="16" height="16" viewBox="0 0 18 18" fill="none">
              <path d="M9 9C10.6569 9 12 7.65685 12 6C12 4.34315 10.6569 3 9 3C7.34315 3 6 4.34315 6 6C6 7.65685 7.34315 9 9 9Z" stroke="currentColor" stroke-width="1.5"/>
              <path d="M15 15C15 12.7909 12.3137 11 9 11C5.68629 11 3 12.7909 3 15" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
            <span>{{ t('profile.profileDetails') }}</span>
          </button>
          <button class="footer-dropdown-item" @mousedown.prevent="openTasks">
            <svg width="16" height="16" viewBox="0 0 18 18" fill="none">
              <path d="M15 3H3C2.44772 3 2 3.44772 2 4V14C2 14.5523 2.44772 15 3 15H15C15.5523 15 16 14.5523 16 14V4C16 3.44772 15.5523 3 15 3Z" stroke="currentColor" stroke-width="1.5"/>
              <path d="M6 7L8 9L12 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            <span>{{ t('profile.myTasks') }}</span>
            <span v-if="pendingTaskCount > 0" class="task-count">{{ pendingTaskCount }}</span>
          </button>
          <div class="footer-dropdown-divider"></div>
          <button class="footer-dropdown-item logout" @mousedown.prevent="handleLogout">
            <svg width="16" height="16" viewBox="0 0 18 18" fill="none">
              <path d="M7 15H4C3.44772 15 3 14.5523 3 14V4C3 3.44772 3.44772 3 4 3H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
              <path d="M11 12L15 9M15 9L11 6M15 9H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            <span>{{ t('profile.logout') }}</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Expand button (collapsed state only) -->
    <button
      v-if="collapsed"
      class="expand-btn"
      @click="collapsed = !collapsed"
      title="Expand sidebar"
    >
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
      </svg>
    </button>
  </aside>
</template>

<script>
import { ref, computed } from 'vue'
import { useAuth } from '../composables/useAuth'
import { useI18n } from '../composables/useI18n'

// Heroicons outline SVGs (24x24 viewBox, stroke="currentColor", stroke-width="1.5")
const OVERVIEW_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M3.75 6A2.25 2.25 0 016 3.75h2.25A2.25 2.25 0 0110.5 6v2.25a2.25 2.25 0 01-2.25 2.25H6a2.25 2.25 0 01-2.25-2.25V6zM3.75 15.75A2.25 2.25 0 016 13.5h2.25a2.25 2.25 0 012.25 2.25V18a2.25 2.25 0 01-2.25 2.25H6A2.25 2.25 0 013.75 18v-2.25zM13.5 6a2.25 2.25 0 012.25-2.25H18A2.25 2.25 0 0120.25 6v2.25A2.25 2.25 0 0118 10.5h-2.25a2.25 2.25 0 01-2.25-2.25V6zM13.5 15.75a2.25 2.25 0 012.25-2.25H18a2.25 2.25 0 012.25 2.25V18A2.25 2.25 0 0118 20.25h-2.25A2.25 2.25 0 0113.5 18v-2.25z" /></svg>`

const INVENTORY_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M20.25 7.5l-.625 10.632a2.25 2.25 0 01-2.247 2.118H6.622a2.25 2.25 0 01-2.247-2.118L3.75 7.5M10 11.25h4M3.375 7.5h17.25c.621 0 1.125-.504 1.125-1.125v-1.5c0-.621-.504-1.125-1.125-1.125H3.375c-.621 0-1.125.504-1.125 1.125v1.5c0 .621.504 1.125 1.125 1.125z" /></svg>`

const ORDERS_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M19.5 14.25v-2.625a3.375 3.375 0 00-3.375-3.375h-1.5A1.125 1.125 0 0113.5 7.125v-1.5a3.375 3.375 0 00-3.375-3.375H8.25m0 12.75h7.5m-7.5 3H12M10.5 2.25H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 00-9-9z" /></svg>`

const FINANCE_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M2.25 18.75a60.07 60.07 0 0115.797 2.101c.727.198 1.453-.342 1.453-1.096V18.75M3.75 4.5v.75A.75.75 0 013 6h-.75m0 0v-.375c0-.621.504-1.125 1.125-1.125H20.25M2.25 6v9m18-10.5v.75c0 .414.336.75.75.75h.75m-1.5-1.5h.375c.621 0 1.125.504 1.125 1.125v9.75c0 .621-.504 1.125-1.125 1.125h-.375m1.5-1.5H21a.75.75 0 00-.75.75v.75m0 0H3.75m0 0h-.375a1.125 1.125 0 01-1.125-1.125V15m1.5 1.5v-.75A.75.75 0 003 15h-.75M15 10.5a3 3 0 11-6 0 3 3 0 016 0zm3 0h.008v.008H18V10.5zm-12 0h.008v.008H6V10.5z" /></svg>`

const DEMAND_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M3 13.125C3 12.504 3.504 12 4.125 12h2.25c.621 0 1.125.504 1.125 1.125v6.75C7.5 20.496 6.996 21 6.375 21h-2.25A1.125 1.125 0 013 19.875v-6.75zM9.75 8.625c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125v11.25c0 .621-.504 1.125-1.125 1.125h-2.25a1.125 1.125 0 01-1.125-1.125V8.625zM16.5 4.125c0-.621.504-1.125 1.125-1.125h2.25C20.496 3 21 3.504 21 4.125v15.75c0 .621-.504 1.125-1.125 1.125h-2.25a1.125 1.125 0 01-1.125-1.125V4.125z" /></svg>`

const REPORTS_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M9 12h3.75M9 15h3.75M9 18h3.75m3 .75H18a2.25 2.25 0 002.25-2.25V6.108c0-1.135-.845-2.098-1.976-2.192a48.424 48.424 0 00-1.123-.08m-5.801 0c-.065.21-.1.433-.1.664 0 .414.336.75.75.75h4.5a.75.75 0 00.75-.75 2.25 2.25 0 00-.1-.664m-5.8 0A2.251 2.251 0 0113.5 2.25H15c1.012 0 1.867.668 2.15 1.586m-5.8 0c-.376.023-.75.05-1.124.08C9.095 4.01 8.25 4.973 8.25 6.108V8.25m0 0H4.875c-.621 0-1.125.504-1.125 1.125v11.25c0 .621.504 1.125 1.125 1.125h9.75c.621 0 1.125-.504 1.125-1.125V9.375c0-.621-.504-1.125-1.125-1.125H8.25zM6.75 12h.008v.008H6.75V12zm0 3h.008v.008H6.75V15zm0 3h.008v.008H6.75V18z" /></svg>`

const RESTOCK_SVG = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0013.803-3.7M4.031 9.865a8.25 8.25 0 0113.803-3.7l3.181 3.182m0-4.991v4.99" /></svg>`

export default {
  name: 'AppSidebar',
  emits: ['show-profile-details', 'show-tasks'],
  setup(props, { emit }) {
    const { currentUser, logout, getInitials } = useAuth()
    const { t, currentLocale, setLocale, availableLocales, localeName } = useI18n()

    const collapsed = ref(false)
    const langMenuOpen = ref(false)
    const profileMenuOpen = ref(false)

    const navItems = computed(() => [
      { path: '/',           label: t('nav.overview'),       icon: OVERVIEW_SVG },
      { path: '/inventory',  label: t('nav.inventory'),      icon: INVENTORY_SVG },
      { path: '/orders',     label: t('nav.orders'),         icon: ORDERS_SVG },
      { path: '/spending',   label: t('nav.finance'),        icon: FINANCE_SVG },
      { path: '/demand',     label: t('nav.demandForecast'), icon: DEMAND_SVG },
      { path: '/reports',    label: 'Reports',               icon: REPORTS_SVG },
      { path: '/restocking', label: t('nav.restocking'),     icon: RESTOCK_SVG },
    ])

    const pendingTaskCount = computed(() =>
      currentUser.value.tasks.filter(task => task.status === 'pending').length
    )

    // Language menu
    const toggleLangMenu = () => {
      langMenuOpen.value = !langMenuOpen.value
      profileMenuOpen.value = false
    }
    const handleLangBlur = () => {
      setTimeout(() => { langMenuOpen.value = false }, 200)
    }
    const selectLanguage = (locale) => {
      setLocale(locale)
      langMenuOpen.value = false
    }
    const languageNames = { en: 'English', ja: '日本語' }
    const getLanguageName = (locale) => languageNames[locale] || locale

    // Profile menu
    const toggleProfileMenu = () => {
      profileMenuOpen.value = !profileMenuOpen.value
      langMenuOpen.value = false
    }
    const handleProfileBlur = () => {
      setTimeout(() => { profileMenuOpen.value = false }, 200)
    }
    const openProfileDetails = () => {
      profileMenuOpen.value = false
      emit('show-profile-details')
    }
    const openTasks = () => {
      profileMenuOpen.value = false
      emit('show-tasks')
    }
    const handleLogout = () => {
      profileMenuOpen.value = false
      logout()
    }

    return {
      t,
      collapsed,
      navItems,
      // language
      langMenuOpen,
      toggleLangMenu,
      handleLangBlur,
      selectLanguage,
      getLanguageName,
      currentLocale,
      availableLocales,
      localeName,
      // profile
      currentUser,
      getInitials,
      pendingTaskCount,
      profileMenuOpen,
      toggleProfileMenu,
      handleProfileBlur,
      openProfileDetails,
      openTasks,
      handleLogout
    }
  }
}
</script>

<style scoped>
/* ── Layout ──────────────────────────────────────────────────────────── */
.sidebar {
  width: var(--sidebar-width);
  flex-shrink: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: var(--sidebar-bg);
  border-right: 1px solid var(--sidebar-border);
  transition: width 0.22s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
  position: relative;
  z-index: 100;
}

.sidebar.collapsed {
  width: var(--sidebar-width-collapsed);
}

/* ── Brand ───────────────────────────────────────────────────────────── */
.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 0 var(--space-3) 0 var(--space-4);
  height: 64px;
  border-bottom: 1px solid var(--sidebar-border);
  flex-shrink: 0;
}

.sidebar.collapsed .sidebar-brand {
  padding: 0;
  justify-content: center;
}

.brand-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  background: var(--color-accent);
  border-radius: 8px;
  color: white;
  flex-shrink: 0;
}

.brand-text {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.brand-name {
  font-size: var(--text-sm);
  font-weight: 700;
  color: #f1f5f9;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: -0.015em;
}

.brand-sub {
  font-size: 0.688rem;
  color: var(--sidebar-text);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-top: 1px;
}

.collapse-btn-brand {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid var(--sidebar-border);
  border-radius: 6px;
  color: var(--sidebar-text);
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
  flex-shrink: 0;
  padding: 0;
}

.collapse-btn-brand:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #cbd5e1;
}

.collapse-btn-brand svg {
  width: 14px;
  height: 14px;
}

/* ── Navigation ──────────────────────────────────────────────────────── */
.sidebar-nav {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: var(--space-4) var(--space-3);
  scrollbar-width: none;
}

.sidebar-nav::-webkit-scrollbar {
  display: none;
}

.nav-section {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

.nav-section-label {
  font-size: 0.688rem;
  font-weight: 600;
  color: #475569;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  padding: 0 var(--space-2);
  margin-bottom: var(--space-1);
  white-space: nowrap;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) 0.625rem;
  border-radius: 7px;
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: var(--text-sm);
  font-weight: 500;
  transition: background 0.15s ease, color 0.15s ease;
  white-space: nowrap;
  overflow: hidden;
}

.nav-link:hover {
  background: var(--sidebar-hover-bg);
  color: #e2e8f0;
}

/* CSS-only active state via Vue Router — no JavaScript path checks */
.nav-link.router-link-exact-active {
  background: var(--sidebar-active-bg);
  color: var(--sidebar-active-text);
}

.nav-link.router-link-exact-active .nav-icon {
  color: var(--sidebar-active-icon);
}

.nav-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  color: inherit;
}

.nav-icon :deep(svg) {
  width: 20px;
  height: 20px;
}

.nav-label {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Collapsed: center icons */
.sidebar.collapsed .nav-link {
  justify-content: center;
  padding: var(--space-2);
}

.sidebar.collapsed .nav-section-label {
  display: none;
}

/* ── Footer ──────────────────────────────────────────────────────────── */
.sidebar-footer {
  border-top: 1px solid var(--sidebar-border);
  padding: var(--space-3);
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  flex-shrink: 0;
}

.footer-divider {
  height: 1px;
  background: var(--sidebar-border);
  margin: var(--space-1) 0;
}

/* Language */
.footer-lang {
  position: relative;
}

.footer-lang-btn {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  width: 100%;
  padding: var(--space-2) 0.625rem;
  background: transparent;
  border: none;
  border-radius: 7px;
  color: var(--sidebar-text);
  cursor: pointer;
  font-family: inherit;
  font-size: 0.813rem;
  font-weight: 500;
  transition: background 0.15s, color 0.15s;
  white-space: nowrap;
  overflow: hidden;
}

.footer-lang-btn:hover {
  background: var(--sidebar-hover-bg);
  color: #e2e8f0;
}

.footer-icon {
  flex-shrink: 0;
}

.footer-btn-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar.collapsed .footer-lang-btn {
  justify-content: center;
  padding: var(--space-2);
}

/* Profile */
.footer-profile {
  position: relative;
}

.footer-profile-btn {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  width: 100%;
  padding: var(--space-2) 0.625rem;
  background: transparent;
  border: none;
  border-radius: 7px;
  color: var(--sidebar-text);
  cursor: pointer;
  font-family: inherit;
  transition: background 0.15s;
  white-space: nowrap;
  overflow: hidden;
}

.footer-profile-btn:hover {
  background: var(--sidebar-hover-bg);
}

.sidebar.collapsed .footer-profile-btn {
  justify-content: center;
  padding: var(--space-2);
}

.avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--color-accent) 0%, #1e40af 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.688rem;
  letter-spacing: 0.025em;
  flex-shrink: 0;
}

.profile-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  text-align: left;
}

.profile-name {
  font-size: 0.813rem;
  font-weight: 600;
  color: #e2e8f0;
  overflow: hidden;
  text-overflow: ellipsis;
}

.profile-role {
  font-size: 0.688rem;
  color: #475569;
  overflow: hidden;
  text-overflow: ellipsis;
}

.chevron {
  color: #475569;
  transition: transform 0.2s ease;
  flex-shrink: 0;
}

.chevron-up {
  transform: rotate(180deg);
}

/* ── Dropdowns ───────────────────────────────────────────────────────── */
.footer-dropdown {
  position: absolute;
  left: calc(100% + var(--space-2));
  bottom: 0;
  background: #1e293b;
  border: 1px solid var(--sidebar-border);
  border-radius: 10px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.4);
  min-width: 220px;
  z-index: 200;
  overflow: hidden;
}

/* Expanded sidebar: open dropdown upward */
.sidebar:not(.collapsed) .footer-dropdown {
  left: 0;
  right: 0;
  bottom: calc(100% + var(--space-2));
  min-width: unset;
}

.lang-dropdown {
  min-width: 160px;
}

.sidebar:not(.collapsed) .lang-dropdown {
  min-width: unset;
}

.footer-dropdown-item {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  width: 100%;
  padding: 0.625rem var(--space-4);
  background: none;
  border: none;
  color: var(--sidebar-text);
  font-family: inherit;
  font-size: 0.813rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
  text-align: left;
}

.footer-dropdown-item:hover {
  background: rgba(255, 255, 255, 0.07);
  color: #e2e8f0;
}

.footer-dropdown-item.active {
  color: var(--sidebar-active-text);
}

.footer-dropdown-item.logout {
  color: #f87171;
}

.footer-dropdown-item.logout:hover {
  background: rgba(239, 68, 68, 0.1);
}

.footer-dropdown-item svg {
  flex-shrink: 0;
  color: inherit;
}

.footer-dropdown-item span:first-of-type {
  flex: 1;
}

.footer-dropdown-divider {
  height: 1px;
  background: var(--sidebar-border);
  margin: var(--space-1) 0;
}

.profile-dropdown-header {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-4);
  background: rgba(255, 255, 255, 0.03);
}

.avatar-lg {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--color-accent) 0%, #1e40af 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: var(--text-sm);
  flex-shrink: 0;
}

.profile-dropdown-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.profile-dropdown-name {
  font-size: var(--text-sm);
  font-weight: 600;
  color: #e2e8f0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.profile-dropdown-email {
  font-size: var(--text-xs);
  color: #475569;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.task-count {
  margin-left: auto;
  background: var(--color-accent);
  color: white;
  font-size: 0.688rem;
  font-weight: 700;
  padding: var(--space-1) 0.4rem;
  border-radius: 10px;
  min-width: 18px;
  text-align: center;
}

/* ── Expand button (collapsed state) ────────────────────────────────── */
.expand-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: var(--space-3) 0;
  background: transparent;
  border: none;
  border-top: 1px solid var(--sidebar-border);
  color: var(--sidebar-text);
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
  flex-shrink: 0;
}

.expand-btn:hover {
  background: var(--sidebar-hover-bg);
  color: #e2e8f0;
}

.expand-btn svg {
  width: 16px;
  height: 16px;
}

/* ── Transitions ─────────────────────────────────────────────────────── */
.fade-label-enter-active,
.fade-label-leave-active {
  transition: opacity 0.15s ease, transform 0.15s ease;
}

.fade-label-enter-from,
.fade-label-leave-to {
  opacity: 0;
  transform: translateX(-4px);
}
</style>
