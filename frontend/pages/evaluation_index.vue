<script setup>
import { useRouter } from 'vue-router'
import { useAuthStore } from '~/stores/auth'

const router = useRouter()
const config = useRuntimeConfig()
const authStore = useAuthStore()

// ดึงข้อมูล
const { data: assignments, pending, error } = await useFetch(`${config.public.apiBase}/api/evaluator/assignments`, {
    headers: { Authorization: `Bearer ${authStore.token}` }
})

// แปลงเป็น Array กัน error
const items = computed(() => assignments.value || [])

// ไปหน้าประเมิน
const selectEvaluatee = (id) => {
    router.push(`/assess-hybrid?id=${id}`)
}

// Helper: สีสถานะ
const getStatusColor = (status) => {
    if(status === 'completed') return 'success'
    if(status === 'in_progress') return 'warning'
    return 'grey'
}

// Helper: ข้อความสถานะ
const getStatusText = (status) => {
    const statusMap = {
        completed: 'เสร็จสิ้น',
        in_progress: 'กำลังดำเนินการ',
        pending: 'รอดำเนินการ'
    }
    return statusMap[status] || 'รอดำเนินการ'
}

// Helper: ไอคอนสถานะ
const getStatusIcon = (status) => {
    const iconMap = {
        completed: 'mdi-check-circle',
        in_progress: 'mdi-clock-outline',
        pending: 'mdi-alert-circle-outline'
    }
    return iconMap[status] || 'mdi-alert-circle-outline'
}
</script>

<template>
  <div class="evaluator-bg" style="min-height: 100vh;">
    <!-- Header Section -->
    <v-sheet class="header-gradient px-6 px-md-12 py-8 mb-n6" elevation="0">
      <v-row align="center" justify="space-between" no-gutters>
        <v-col cols="12" md="auto">
          <div class="d-flex align-center mb-2">
            <v-avatar color="white" size="56" class="mr-3 elevation-3">
              <v-icon color="primary" size="32">mdi-clipboard-account</v-icon>
            </v-avatar>
            <div>
              <h1 class="text-h4 text-md-h3 font-weight-black text-white mb-1">
                รายการประเมินบุคลากร
              </h1>
              <div class="text-subtitle-1 text-white" style="opacity: 0.9">
                เลือกบุคลากรเพื่อดำเนินการประเมิน
              </div>
            </div>
          </div>
        </v-col>
        <v-col cols="12" md="auto" class="text-center text-md-right mt-4 mt-md-0">
          <v-chip color="white" size="large" variant="flat" class="font-weight-bold">
            <v-icon start color="primary">mdi-account-multiple</v-icon>
            ทั้งหมด {{ items.length }} รายการ
          </v-chip>
        </v-col>
      </v-row>
    </v-sheet>

    <!-- Main Content -->
    <v-container fluid class="px-6 px-md-12 pt-12 pb-8">
      <!-- Loading State -->
      <div v-if="pending" class="text-center py-16">
        <v-progress-circular 
          indeterminate 
          color="primary" 
          size="64" 
          width="6"
        ></v-progress-circular>
        <div class="text-h6 text-grey-darken-1 mt-4">กำลังโหลดข้อมูล...</div>
      </div>

      <!-- Error State -->
      <v-card v-else-if="error" elevation="4" rounded="xl" class="pa-8 text-center">
        <v-avatar color="error" size="80" class="mb-4">
          <v-icon color="white" size="48">mdi-alert-circle</v-icon>
        </v-avatar>
        <div class="text-h6 text-error font-weight-bold mb-2">
          เกิดข้อผิดพลาดในการโหลดข้อมูล
        </div>
        <div class="text-body-2 text-grey-darken-1">
          {{ error.message }}
        </div>
      </v-card>

      <!-- Empty State -->
      <v-card v-else-if="items.length === 0" elevation="4" rounded="xl" class="pa-12 text-center">
        <v-avatar color="grey-lighten-3" size="120" class="mb-6">
          <v-icon color="grey-lighten-1" size="72">mdi-clipboard-text-off</v-icon>
        </v-avatar>
        <div class="text-h5 text-grey-darken-2 font-weight-bold mb-2">
          ไม่มีรายการที่ต้องประเมิน
        </div>
        <div class="text-body-1 text-grey">
          ขณะนี้ยังไม่มีรายการประเมินที่มอบหมายให้คุณ
        </div>
      </v-card>

      <!-- Cards Grid -->
      <v-row v-else>
        <v-col v-for="item in items" :key="item.id" cols="12" sm="6" md="4" lg="3">
          <v-card 
            hover 
            @click="selectEvaluatee(item.id)" 
            class="evaluatee-card h-100 rounded-xl"
            elevation="6"
          >
            <!-- Status Header -->
            <div class="status-header pa-3 d-flex justify-space-between align-center">
              <v-chip 
                :color="getStatusColor(item.status)" 
                size="small" 
                variant="flat"
                class="font-weight-bold"
              >
                <v-icon start size="16">{{ getStatusIcon(item.status) }}</v-icon>
                {{ getStatusText(item.status) }}
              </v-chip>
              <v-icon color="white" size="20">mdi-chevron-right</v-icon>
            </div>

            <!-- Card Content -->
            <v-card-text class="text-center pt-6 pb-6">
              <!-- Avatar -->
              <div class="avatar-wrapper mb-4">
                <v-avatar 
                  size="96" 
                  color="primary" 
                  class="elevation-4 avatar-main"
                >
                  <span class="text-h3 font-weight-black text-white">
                    {{ item.evaluatee_name?.charAt(0) || '?' }}
                  </span>
                </v-avatar>
                <v-avatar 
                  size="32" 
                  color="success" 
                  class="elevation-2 avatar-badge"
                  v-if="item.status === 'completed'"
                >
                  <v-icon color="white" size="20">mdi-check</v-icon>
                </v-avatar>
              </div>

              <!-- Name & Info -->
              <div class="mb-4">
                <div class="text-h6 font-weight-black text-grey-darken-3 mb-2">
                  {{ item.evaluatee_name }}
                </div>
                <v-chip 
                  color="grey-lighten-3" 
                  size="small" 
                  variant="flat"
                  class="mb-2"
                >
                  <v-icon start size="14">mdi-account-tie</v-icon>
                  {{ item.position }}
                </v-chip>
                <div class="text-body-2 text-grey-darken-1">
                  <v-icon size="14" class="mr-1">mdi-domain</v-icon>
                  {{ item.department }}
                </div>
              </div>

              <!-- Period -->
              <v-divider class="mb-3"></v-divider>
              <div class="text-caption text-primary font-weight-bold">
                <v-icon size="14" class="mr-1">mdi-calendar-clock</v-icon>
                รอบ: {{ item.period_name }}
              </div>
            </v-card-text>

            <!-- Action Footer -->
            <v-card-actions class="pa-4 pt-0">
              <v-btn 
                block 
                color="primary" 
                variant="flat" 
                rounded="lg"
                size="large"
                class="font-weight-bold action-btn"
              >
                <v-icon start>mdi-clipboard-edit</v-icon>
                {{ item.status === 'completed' ? 'ดูรายละเอียด' : 'เริ่มประเมิน' }}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<style scoped>
/* Background - เหมือนหน้า Dashboard */
.evaluator-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Header Gradient - เหมือนหน้า Dashboard */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

/* Status Header */
.status-header {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.9) 0%, rgba(118, 75, 162, 0.9) 100%);
  border-radius: 16px 16px 0 0;
}

/* Evaluatee Card */
.evaluatee-card {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  border: 2px solid rgba(102, 126, 234, 0.1);
  background: white;
}

.evaluatee-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 16px 40px rgba(102, 126, 234, 0.25) !important;
  border-color: rgba(102, 126, 234, 0.3);
}

/* Avatar Wrapper */
.avatar-wrapper {
  position: relative;
  display: inline-block;
}

.avatar-main {
  border: 4px solid white;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  transition: transform 0.3s ease;
}

.evaluatee-card:hover .avatar-main {
  transform: scale(1.1);
}

.avatar-badge {
  position: absolute;
  bottom: 0;
  right: 0;
  border: 3px solid white;
}

/* Action Button */
.action-btn {
  transition: all 0.3s ease;
}

.evaluatee-card:hover .action-btn {
  transform: scale(1.05);
}

/* Loading Animation */
:deep(.v-progress-circular) {
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
}

/* Responsive */
@media (max-width: 960px) {
  .avatar-main {
    width: 80px !important;
    height: 80px !important;
  }
  
  .avatar-main span {
    font-size: 2rem !important;
  }
}
</style>