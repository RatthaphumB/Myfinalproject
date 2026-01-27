<script setup>
import { ref, onMounted } from 'vue'
import { useAuthStore } from '~/stores/auth'

const auth = useAuthStore()
const stats = ref({ uploads: 0, evaluations: 0, score: 0 })
const loading = ref(true)

onMounted(async () => {
  // โหลดข้อมูลตัวอย่าง
  setTimeout(() => {
    stats.value = { uploads: 12, evaluations: 5, score: 88 }
    loading.value = false
  }, 800)
})

function logout() {
  auth.logout()
  navigateTo('/login')
}

definePageMeta({
  middleware: 'auth'
})
</script>

<template>
  <v-app>
    <v-main class="dashboard-bg">
      <v-container fluid class="pa-0">
        <!-- Header Section -->
        <v-sheet class="header-gradient px-6 px-md-12 py-8 py-md-10" elevation="0">
          <v-row align="center" justify="space-between" no-gutters>
            <v-col cols="12" md="auto" class="mb-4 mb-md-0">
              <div class="d-flex align-center mb-2">
                <v-avatar color="white" size="48" class="mr-3 elevation-3">
                  <v-icon color="primary" size="28">mdi-account-circle</v-icon>
                </v-avatar>
                <div>
                  <div class="text-h5 text-md-h4 font-weight-bold text-white mb-1">
                    สวัสดี, {{ auth.user?.name || 'ผู้ใช้' }} 👋
                  </div>
                  <div class="text-subtitle-1 text-white" style="opacity: 0.9">
                    ยินดีต้อนรับเข้าสู่ระบบจัดการข้อมูล VEC Skills
                  </div>
                </div>
              </div>
            </v-col>
            <v-col cols="12" md="auto" class="text-center text-md-right">
              <v-btn
                prepend-icon="mdi-logout"
                variant="outlined"
                color="white"
                size="large"
                rounded="lg"
                class="font-weight-medium"
                @click="logout"
              >
                ออกจากระบบ
              </v-btn>
            </v-col>
          </v-row>
        </v-sheet>

        <!-- Stats Cards Section -->
        <v-container class="mt-n8 px-6 px-md-12 pb-12">
          <v-row v-if="!loading">
            <!-- Card 1: Uploads -->
            <v-col cols="12" md="4">
              <v-card
                class="stat-card rounded-xl pa-6 elevation-4"
                hover
              >
                <div class="d-flex align-center justify-space-between mb-4">
                  <v-avatar color="indigo-lighten-5" size="64" rounded="lg">
                    <v-icon color="indigo-darken-1" size="36">mdi-file-document-multiple</v-icon>
                  </v-avatar>
                  <v-chip color="indigo-lighten-4" variant="flat" size="small">
                    <v-icon start size="16">mdi-trending-up</v-icon>
                    +2
                  </v-chip>
                </div>
                <div class="text-h3 font-weight-black text-indigo-darken-2 mb-2">
                  {{ stats.uploads }}
                </div>
                <div class="text-subtitle-1 text-grey-darken-1 font-weight-medium">
                  ไฟล์หลักฐานทั้งหมด
                </div>
                <v-progress-linear
                  color="indigo"
                  model-value="75"
                  height="6"
                  rounded
                  class="mt-4"
                />
              </v-card>
            </v-col>

            <!-- Card 2: Evaluations -->
            <v-col cols="12" md="4">
              <v-card
                class="stat-card rounded-xl pa-6 elevation-4"
                hover
              >
                <div class="d-flex align-center justify-space-between mb-4">
                  <v-avatar color="green-lighten-5" size="64" rounded="lg">
                    <v-icon color="green-darken-1" size="36">mdi-clipboard-check</v-icon>
                  </v-avatar>
                  <v-chip color="green-lighten-4" variant="flat" size="small">
                    <v-icon start size="16">mdi-check-circle</v-icon>
                    สำเร็จ
                  </v-chip>
                </div>
                <div class="text-h3 font-weight-black text-green-darken-2 mb-2">
                  {{ stats.evaluations }}
                </div>
                <div class="text-subtitle-1 text-grey-darken-1 font-weight-medium">
                  การประเมินที่สำเร็จ
                </div>
                <v-progress-linear
                  color="green"
                  model-value="42"
                  height="6"
                  rounded
                  class="mt-4"
                />
              </v-card>
            </v-col>

            <!-- Card 3: Score -->
            <v-col cols="12" md="4">
              <v-card
                class="stat-card rounded-xl pa-6 elevation-4"
                hover
              >
                <div class="d-flex align-center justify-space-between mb-4">
                  <v-avatar color="orange-lighten-5" size="64" rounded="lg">
                    <v-icon color="orange-darken-1" size="36">mdi-trophy</v-icon>
                  </v-avatar>
                  <v-chip color="orange-lighten-4" variant="flat" size="small">
                    <v-icon start size="16">mdi-star</v-icon>
                    ดีมาก
                  </v-chip>
                </div>
                <div class="text-h3 font-weight-black text-orange-darken-2 mb-2">
                  {{ stats.score }}%
                </div>
                <div class="text-subtitle-1 text-grey-darken-1 font-weight-medium">
                  คะแนนเฉลี่ยรวม
                </div>
                <v-progress-linear
                  color="orange"
                  :model-value="stats.score"
                  height="6"
                  rounded
                  class="mt-4"
                />
              </v-card>
            </v-col>
          </v-row>

          <!-- Loading State -->
          <v-row v-else>
            <v-col v-for="i in 3" :key="i" cols="12" md="4">
              <v-skeleton-loader
                type="image, list-item-two-line"
                class="rounded-xl elevation-4"
                height="220"
              />
            </v-col>
          </v-row>

          <!-- Quick Actions Section -->
          <v-row class="mt-8" v-if="!loading">
            <v-col cols="12">
              <v-card class="rounded-xl elevation-2 pa-6">
                <div class="text-h6 font-weight-bold text-grey-darken-2 mb-6">
                  <v-icon class="mr-2" color="primary">mdi-lightning-bolt</v-icon>
                  การดำเนินการด่วน
                </div>
                <v-row>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      size="large"
                      color="primary"
                      variant="flat"
                      rounded="lg"
                      prepend-icon="mdi-upload"
                      class="text-none font-weight-medium"
                    >
                      อัปโหลดไฟล์
                    </v-btn>
                  </v-col>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      size="large"
                      color="success"
                      variant="flat"
                      rounded="lg"
                      prepend-icon="mdi-clipboard-text"
                      class="text-none font-weight-medium"
                    >
                      ประเมินผล
                    </v-btn>
                  </v-col>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      size="large"
                      color="info"
                      variant="flat"
                      rounded="lg"
                      prepend-icon="mdi-chart-line"
                      class="text-none font-weight-medium"
                    >
                      รายงาน
                    </v-btn>
                  </v-col>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      size="large"
                      color="warning"
                      variant="flat"
                      rounded="lg"
                      prepend-icon="mdi-cog"
                      class="text-none font-weight-medium"
                    >
                      ตั้งค่า
                    </v-btn>
                  </v-col>
                </v-row>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-container>
    </v-main>

    <v-footer class="footer-gradient" app border="t-lg">
      <v-container class="py-4">
        <div class="text-center">
          <div class="text-body-2 text-white mb-1">
            &copy; {{ new Date().getFullYear() }} <strong>VEC Skills</strong>
          </div>
          <div class="text-caption text-white" style="opacity: 0.85">
            <v-icon size="16" class="mr-1">mdi-lightning-bolt</v-icon>
            Powered by <span class="font-weight-bold">KruOak</span>
          </div>
        </div>
      </v-container>
    </v-footer>
  </v-app>
</template>

<style scoped>
.dashboard-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
  min-height: 100vh;
}

.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

.footer-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 -4px 20px rgba(102, 126, 234, 0.2);
}

.stat-card {
  background: white;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(0, 0, 0, 0.05);
}

.stat-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12) !important;
}

.v-progress-linear {
  border-radius: 8px;
}

.v-chip {
  font-weight: 600;
}
</style>