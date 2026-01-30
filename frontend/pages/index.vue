<script setup>
import { ref, onMounted } from 'vue'
import { useAuthStore } from '~/stores/auth'
// import { useDisplay } from 'vuetify' // ถ้าต้องการใช้ logic ตรวจสอบขนาดหน้าจอ

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
        
        <v-sheet class="header-gradient px-4 px-md-12 py-6 py-md-10 rounded-b-xl" elevation="0">
          <v-row align="center" justify="space-between" no-gutters>
            <v-col cols="12" md="auto" class="mb-4 mb-md-0 text-center text-md-left">
              <div class="d-flex flex-column flex-md-row align-center">
                <v-avatar color="white" size="56" class="mb-3 mb-md-0 mr-md-4 elevation-3 border-opacity-25">
                  <v-icon color="primary" size="32">mdi-account-circle</v-icon>
                </v-avatar>
                
                <div>
                  <div class="text-h5 text-md-h4 font-weight-bold text-white mb-1">
                    สวัสดี, {{ auth.user?.name || 'ผู้ใช้' }} 👋
                  </div>
                  <div class="text-body-2 text-subtitle-1 text-white opacity-90">
                    ยินดีต้อนรับเข้าสู่ระบบจัดการข้อมูล VEC Skills
                  </div>
                </div>
              </div>
            </v-col>

            <v-col cols="12" md="auto" class="text-center text-md-right mt-2 mt-md-0">
              <v-btn
                prepend-icon="mdi-logout"
                variant="outlined"
                color="white"
                size="large"
                rounded="lg"
                class="font-weight-medium px-6"
                :block="$vuetify.display.smAndDown"
                @click="logout"
              >
                ออกจากระบบ
              </v-btn>
            </v-col>
          </v-row>
        </v-sheet>

        <v-container class="mt-n6 mt-md-n10 px-4 px-md-12 pb-12 position-relative" style="z-index: 2;">
          <v-row v-if="!loading">
            <v-col cols="12" sm="6" md="4">
              <v-card class="stat-card rounded-xl pa-5 pa-md-6 elevation-3" hover>
                <div class="d-flex align-center justify-space-between mb-4">
                  <v-avatar color="indigo-lighten-5" size="56" rounded="lg">
                    <v-icon color="indigo-darken-1" size="30">mdi-file-document-multiple</v-icon>
                  </v-avatar>
                  <v-chip color="indigo-lighten-4" variant="flat" size="small" class="font-weight-bold">
                    <v-icon start size="14">mdi-trending-up</v-icon>
                    +2
                  </v-chip>
                </div>
                <div class="text-h4 text-md-h3 font-weight-black text-indigo-darken-2 mb-1">
                  {{ stats.uploads }}
                </div>
                <div class="text-body-2 text-subtitle-1 text-grey-darken-1 font-weight-medium">
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

            <v-col cols="12" sm="6" md="4">
              <v-card class="stat-card rounded-xl pa-5 pa-md-6 elevation-3" hover>
                <div class="d-flex align-center justify-space-between mb-4">
                  <v-avatar color="green-lighten-5" size="56" rounded="lg">
                    <v-icon color="green-darken-1" size="30">mdi-clipboard-check</v-icon>
                  </v-avatar>
                  <v-chip color="green-lighten-4" variant="flat" size="small" class="font-weight-bold">
                    <v-icon start size="14">mdi-check-circle</v-icon>
                    สำเร็จ
                  </v-chip>
                </div>
                <div class="text-h4 text-md-h3 font-weight-black text-green-darken-2 mb-1">
                  {{ stats.evaluations }}
                </div>
                <div class="text-body-2 text-subtitle-1 text-grey-darken-1 font-weight-medium">
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

            <v-col cols="12" md="4">
              <v-card class="stat-card rounded-xl pa-5 pa-md-6 elevation-3" hover>
                <div class="d-flex align-center justify-space-between mb-4">
                  <v-avatar color="orange-lighten-5" size="56" rounded="lg">
                    <v-icon color="orange-darken-1" size="30">mdi-trophy</v-icon>
                  </v-avatar>
                  <v-chip color="orange-lighten-4" variant="flat" size="small" class="font-weight-bold">
                    <v-icon start size="14">mdi-star</v-icon>
                    ดีมาก
                  </v-chip>
                </div>
                <div class="text-h4 text-md-h3 font-weight-black text-orange-darken-2 mb-1">
                  {{ stats.score }}%
                </div>
                <div class="text-body-2 text-subtitle-1 text-grey-darken-1 font-weight-medium">
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

          <v-row v-else>
            <v-col v-for="i in 3" :key="i" cols="12" md="4">
              <v-skeleton-loader
                type="image, list-item-two-line"
                class="rounded-xl elevation-3"
                height="200"
              />
            </v-col>
          </v-row>

          <v-row class="mt-4 mt-md-8" v-if="!loading">
            <v-col cols="12">
              <v-card class="rounded-xl elevation-2 pa-5 pa-md-6 border">
                <div class="text-h6 font-weight-bold text-grey-darken-2 mb-4 mb-md-6 d-flex align-center">
                  <v-icon class="mr-2" color="primary">mdi-lightning-bolt</v-icon>
                  การดำเนินการด่วน
                </div>
                <v-row dense>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      height="60"
                      color="primary"
                      variant="flat"
                      rounded="lg"
                      class="text-none font-weight-medium d-flex flex-column align-center justify-center pt-2 pb-2"
                    >
                      <v-icon size="24" class="mb-1">mdi-upload</v-icon>
                      <span class="text-caption text-md-body-2">อัปโหลด</span>
                    </v-btn>
                  </v-col>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      height="60"
                      color="success"
                      variant="flat"
                      rounded="lg"
                      class="text-none font-weight-medium d-flex flex-column align-center justify-center pt-2 pb-2"
                    >
                      <v-icon size="24" class="mb-1">mdi-clipboard-text</v-icon>
                      <span class="text-caption text-md-body-2">ประเมินผล</span>
                    </v-btn>
                  </v-col>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      height="60"
                      color="info"
                      variant="flat"
                      rounded="lg"
                      class="text-none font-weight-medium d-flex flex-column align-center justify-center pt-2 pb-2"
                    >
                      <v-icon size="24" class="mb-1">mdi-chart-line</v-icon>
                      <span class="text-caption text-md-body-2">รายงาน</span>
                    </v-btn>
                  </v-col>
                  <v-col cols="6" md="3">
                    <v-btn
                      block
                      height="60"
                      color="warning"
                      variant="flat"
                      rounded="lg"
                      class="text-none font-weight-medium d-flex flex-column align-center justify-center pt-2 pb-2"
                    >
                      <v-icon size="24" class="mb-1">mdi-cog</v-icon>
                      <span class="text-caption text-md-body-2">ตั้งค่า</span>
                    </v-btn>
                  </v-col>
                </v-row>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-container>
    </v-main>

    <v-footer class="footer-gradient" app border="t-lg" absolute>
      <v-container class="py-4">
        <div class="text-center">
          <div class="text-caption text-md-body-2 text-white mb-1">
            &copy; {{ new Date().getFullYear() }} <strong>VEC Skills</strong>
          </div>
          <div class="text-caption text-white" style="opacity: 0.7">
            Designed for Excellence
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
  /* เพิ่มความโค้งมนด้านล่างเพื่อให้ดูทันสมัยขึ้นบนมือถือ */
  border-bottom-left-radius: 24px !important;
  border-bottom-right-radius: 24px !important;
}

.footer-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  /* ยกเลิก shadow ด้านล่างถ้าไม่จำเป็น */
}

.stat-card {
  background: white;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(0, 0, 0, 0.05);
  /* ป้องกันการ overflow บนหน้าจอเล็ก */
  overflow: hidden; 
}

.stat-card:hover {
  transform: translateY(-5px); /* ลดระยะการเด้งลงเล็กน้อยบนมือถือ */
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1) !important;
}

/* ปรับแต่งปุ่มกด Action ให้มี icon กับ text ซ้อนกันสวยงาม */
.v-btn--variant-flat {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.opacity-90 {
  opacity: 0.9;
}
</style>