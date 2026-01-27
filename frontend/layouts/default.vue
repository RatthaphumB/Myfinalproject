<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useMenu } from '~/composables/useMenu'
import { useAuthStore } from '~/stores/auth'

const router = useRouter()
const drawer = ref(true)
const authStore = useAuthStore()

// ดึง Role จาก Store
const role = computed(() => authStore.user?.role || '')

// เรียกใช้ Composable
const { menu } = useMenu(role)

// ข้อมูลโปรไฟล์
const userProfile = computed(() => ({
  name: authStore.user?.name_th || 'Guest',
  email: authStore.user?.email || '',
  initials: (authStore.user?.name_th || 'G').charAt(0),
  role: authStore.user?.role || 'user'
}))

// Badge สีตาม Role
const roleBadge = computed(() => {
  const badges = {
    admin: { text: 'ผู้ดูแลระบบ', color: 'error', icon: 'mdi-shield-crown' },
    evaluator: { text: 'กรรมการ', color: 'primary', icon: 'mdi-account-tie' },
    evaluatee: { text: 'ผู้รับการประเมิน', color: 'warning', icon: 'mdi-account-school' }
  }
  return badges[userProfile.value.role] || { text: 'ผู้ใช้', color: 'grey', icon: 'mdi-account' }
})

onMounted(() => {
  authStore.hydrateFromStorage()
})

// ฟังก์ชัน Logout
const logout = () => {
  authStore.logout()
  router.push('/login')
}
</script>

<template>
  <v-app>
    <!-- Navigation Drawer with Gradient -->
    <v-navigation-drawer 
      v-model="drawer" 
      width="280" 
      elevation="0" 
      class="drawer-gradient"
    >
      
      <!-- Header Logo Section -->
      <div class="pa-6 drawer-header">
        <div class="d-flex align-center mb-4">
          <v-avatar color="white" size="56" class="mr-3 elevation-4" rounded="lg">
            <v-icon icon="mdi-shield-star" color="primary" size="32"></v-icon>
          </v-avatar>
          <div>
            <div class="text-h6 font-weight-black text-white">VEC Skills</div>
            <div class="text-caption text-white" style="opacity: 0.85">Evaluation System</div>
          </div>
        </div>

        <!-- User Profile Card -->
        <v-card class="profile-card rounded-xl" elevation="4">
          <v-card-text class="pa-4">
            <div class="d-flex align-center mb-2">
              <v-avatar :color="roleBadge.color" size="48" class="mr-3">
                <span class="text-h6 font-weight-bold text-white">{{ userProfile.initials }}</span>
              </v-avatar>
              <div class="flex-grow-1">
                <div class="text-subtitle-2 font-weight-bold text-grey-darken-3">
                  {{ userProfile.name }}
                </div>
                <v-chip 
                  :color="roleBadge.color" 
                  size="x-small" 
                  variant="flat"
                  class="mt-1"
                >
                  <v-icon start size="12">{{ roleBadge.icon }}</v-icon>
                  {{ roleBadge.text }}
                </v-chip>
              </div>
            </div>
          </v-card-text>
        </v-card>
      </div>

      <v-divider class="my-2" style="opacity: 0.2"></v-divider>

      <!-- Menu List -->
      <v-list density="comfortable" nav class="px-3 menu-list">
        <template v-for="(section, i) in menu" :key="i">
          <div 
            v-if="section.label" 
            class="px-3 py-3 text-caption text-uppercase font-weight-bold mt-2 section-label"
          >
            <v-icon size="14" class="mr-1">mdi-menu</v-icon>
            {{ section.label }}
          </div>
          
          <v-list-item
            v-for="(it, j) in section.items"
            :key="j"
            :to="it.to"
            :prepend-icon="it.icon"
            :title="it.label"
            active-color="primary"
            rounded="xl"
            class="mb-1 menu-item"
          />
        </template>
      </v-list>

      <!-- Logout Button at Bottom -->
      <template v-slot:append>
        <div class="pa-4">
          <v-btn 
            block 
            color="white" 
            variant="flat" 
            prepend-icon="mdi-logout" 
            rounded="lg"
            size="large"
            class="logout-btn"
            @click="logout"
          >
            <span class="font-weight-bold">ออกจากระบบ</span>
          </v-btn>
        </div>
      </template>
    </v-navigation-drawer>

    <!-- App Bar -->
    <v-app-bar color="white" elevation="2" height="72">
      <v-app-bar-nav-icon 
        @click="drawer = !drawer" 
        class="ml-2"
        size="large"
      >
        <v-icon>mdi-menu</v-icon>
      </v-app-bar-nav-icon>
      
      <v-toolbar-title class="ml-2">
        <div class="d-flex align-center">
          <v-icon color="primary" size="28" class="mr-2">mdi-chart-box</v-icon>
          <div>
            <div class="text-subtitle-1 font-weight-bold text-grey-darken-3">
              ระบบประเมินบุคลากร
            </div>
            <div class="text-caption text-grey">VEC Skills Management</div>
          </div>
        </div>
      </v-toolbar-title>

      <v-spacer />

      <!-- Notification Button -->
      <v-btn icon variant="text" class="mr-2">
        <v-badge color="error" content="3" floating>
          <v-icon>mdi-bell-outline</v-icon>
        </v-badge>
      </v-btn>

      <!-- User Menu -->
      <v-menu location="bottom end" transition="scale-transition" offset="12">
        <template v-slot:activator="{ props }">
          <v-btn 
            v-bind="props" 
            class="mr-4 user-menu-btn"
            variant="flat"
            rounded="lg"
          >
            <v-avatar :color="roleBadge.color" size="40" class="mr-2">
              <span class="text-subtitle-2 font-weight-bold text-white">{{ userProfile.initials }}</span>
            </v-avatar>
            <div class="text-left d-none d-md-block">
              <div class="text-caption font-weight-bold text-grey-darken-3">{{ userProfile.name }}</div>
              <div class="text-caption text-grey" style="font-size: 0.65rem;">{{ roleBadge.text }}</div>
            </div>
            <v-icon class="ml-2">mdi-chevron-down</v-icon>
          </v-btn>
        </template>

        <v-card width="280" elevation="12" rounded="xl" class="overflow-hidden">
          <!-- Profile Header -->
          <div class="profile-menu-header pa-4">
            <div class="d-flex align-center mb-3">
              <v-avatar :color="roleBadge.color" size="56" class="mr-3">
                <span class="text-h6 font-weight-bold text-white">{{ userProfile.initials }}</span>
              </v-avatar>
              <div>
                <div class="text-subtitle-1 font-weight-bold text-white">
                  {{ userProfile.name }}
                </div>
                <div class="text-caption text-white" style="opacity: 0.9">
                  {{ userProfile.email }}
                </div>
              </div>
            </div>
            <v-chip 
              :color="roleBadge.color" 
              size="small" 
              variant="flat"
            >
              <v-icon start size="14">{{ roleBadge.icon }}</v-icon>
              {{ roleBadge.text }}
            </v-chip>
          </div>

          <v-divider></v-divider>

          <!-- Menu Items -->
          <v-list density="comfortable" class="py-2">
            <v-list-item 
              prepend-icon="mdi-account-circle" 
              title="โปรไฟล์ของฉัน" 
              subtitle="จัดการข้อมูลส่วนตัว"
              rounded="lg"
              class="mx-2"
            ></v-list-item>
            <v-list-item 
              prepend-icon="mdi-cog" 
              title="ตั้งค่า" 
              subtitle="ปรับแต่งระบบ"
              rounded="lg"
              class="mx-2"
            ></v-list-item>
          </v-list>

          <v-divider></v-divider>

          <!-- Logout -->
          <div class="pa-3">
            <v-btn
              block
              color="error"
              variant="flat"
              prepend-icon="mdi-logout"
              rounded="lg"
              class="font-weight-bold"
              @click="logout"
            >
              ออกจากระบบ
            </v-btn>
          </div>
        </v-card>
      </v-menu>
    </v-app-bar>

    <!-- Main Content -->
    <v-main class="main-bg">
      <v-container fluid class="py-6 px-6">
        <slot /> 
      </v-container>
      
      <!-- Footer -->
      <v-footer color="transparent" class="footer-content text-center py-4">
        <v-container>
          <div class="text-body-2 text-grey-darken-1 mb-1">
            &copy; {{ new Date().getFullYear() }} <strong>VEC Skills</strong> — Evaluation System
          </div>
          <div class="text-caption text-grey">
            <v-icon size="14" class="mr-1">mdi-lightning-bolt</v-icon>
            Powered by <span class="font-weight-bold">KruOak</span>
          </div>
        </v-container>
      </v-footer>
    </v-main>
  </v-app>
</template>

<style scoped>
/* Drawer Gradient Background */
.drawer-gradient {
  background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
  border-right: none !important;
}

.drawer-header {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0.05) 100%);
  backdrop-filter: blur(10px);
}

/* Profile Card in Drawer */
.profile-card {
  background: white;
  transition: transform 0.2s ease;
}

.profile-card:hover {
  transform: translateY(-2px);
}

/* Section Label in Menu */
.section-label {
  color: rgba(255, 255, 255, 0.7);
  letter-spacing: 1px;
}

/* Menu Items */
.menu-list :deep(.v-list-item) {
  color: rgba(255, 255, 255, 0.85) !important;
  margin-bottom: 4px;
}

.menu-list :deep(.v-list-item:hover) {
  background-color: rgba(255, 255, 255, 0.1) !important;
}

.menu-list :deep(.v-list-item--active) {
  background-color: white !important;
  color: #667eea !important;
}

.menu-list :deep(.v-list-item--active .v-icon) {
  color: #667eea !important;
}

.menu-list :deep(.v-list-item__prepend > .v-icon) {
  color: rgba(255, 255, 255, 0.85);
}

.menu-list :deep(.v-list-item--active .v-list-item__prepend > .v-icon) {
  color: #667eea !important;
}

/* Logout Button */
.logout-btn {
  color: #667eea !important;
  transition: all 0.3s ease;
}

.logout-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(255, 255, 255, 0.3);
}

/* Main Background */
.main-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* App Bar */
:deep(.v-app-bar) {
  border-bottom: 1px solid rgba(102, 126, 234, 0.1);
}

/* User Menu Button */
.user-menu-btn {
  background: rgba(102, 126, 234, 0.05) !important;
  transition: all 0.2s ease;
}

.user-menu-btn:hover {
  background: rgba(102, 126, 234, 0.1) !important;
}

/* Profile Menu Header */
.profile-menu-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

/* Footer */
.footer-content {
  background: transparent;
  margin-top: 40px;
}

/* Scrollbar for Menu */
.v-navigation-drawer__content {
  scrollbar-width: thin;
  scrollbar-color: rgba(255, 255, 255, 0.3) transparent;
}

.v-navigation-drawer__content::-webkit-scrollbar {
  width: 6px;
}

.v-navigation-drawer__content::-webkit-scrollbar-track {
  background: transparent;
}

.v-navigation-drawer__content::-webkit-scrollbar-thumb {
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

.v-navigation-drawer__content::-webkit-scrollbar-thumb:hover {
  background-color: rgba(255, 255, 255, 0.5);
}

/* Badge Animation */
:deep(.v-badge__badge) {
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.7;
  }
}
</style>