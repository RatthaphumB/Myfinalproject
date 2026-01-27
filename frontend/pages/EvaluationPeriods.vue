<template>
  <div class="periods-bg" style="min-height: 100vh;">
    <!-- Header Section -->
    <v-sheet class="header-gradient px-6 px-md-12 py-8 mb-n6" elevation="0">
      <v-row align="center" justify="space-between" no-gutters>
        <v-col cols="12" md="auto" class="mb-4 mb-md-0">
          <div class="d-flex align-center mb-2">
            <v-avatar color="white" size="56" class="mr-3 elevation-3">
              <v-icon color="primary" size="32">mdi-calendar-clock</v-icon>
            </v-avatar>
            <div>
              <h1 class="text-h4 text-md-h3 font-weight-black text-white mb-1">
                ระบบจัดการรอบการประเมิน
              </h1>
              <div class="text-subtitle-1 text-white" style="opacity: 0.9">
                จัดการช่วงเวลาการส่งหลักฐานและเกณฑ์การให้คะแนน
              </div>
            </div>
          </div>
        </v-col>
      </v-row>
    </v-sheet>

    <!-- Main Content -->
    <v-container fluid class="px-6 px-md-12 pt-12 pb-8">
      <v-card elevation="8" rounded="xl" class="overflow-hidden" style="border: 1px solid rgba(102, 126, 234, 0.1);">
        <!-- Toolbar -->
        <v-toolbar color="white" flat height="80" class="px-4">
          <v-text-field
            v-model="search"
            prepend-inner-icon="mdi-magnify"
            label="ค้นหารอบการประเมิน..."
            variant="outlined"
            flat
            hide-details
            density="comfortable"
            rounded="lg"
            class="search-field"
            style="max-width: 350px;"
          ></v-text-field>
          <v-spacer></v-spacer>
          <v-btn 
            color="primary" 
            variant="flat" 
            prepend-icon="mdi-plus-circle" 
            rounded="lg"
            size="large"
            class="text-none font-weight-bold btn-hover px-6"
            elevation="2"
            @click="openDialog()"
          >
            เปิดรอบประเมินใหม่
          </v-btn>
        </v-toolbar>

        <v-divider></v-divider>

        <!-- Data Table -->
        <v-data-table 
          :headers="headers" 
          :items="periods" 
          :loading="loading"
          :search="search"
          hover 
          class="elevation-0"
        >
          <template v-slot:item.code="{ item }">
            <v-chip
              color="primary"
              variant="flat"
              size="default"
              class="font-weight-bold"
            >
              <v-icon start size="16">mdi-tag</v-icon>
              {{ item.code }}
            </v-chip>
          </template>

          <template v-slot:item.buddhist_year="{ item }">
            <span class="text-h6 font-weight-bold text-primary">{{ item.buddhist_year }}</span>
          </template>

          <template v-slot:item.name_th="{ item }">
            <div class="py-3">
              <div class="text-subtitle-1 font-weight-bold text-grey-darken-3">
                {{ item.name_th }}
              </div>
            </div>
          </template>

          <template v-slot:item.period_date="{ item }">
            <v-card variant="tonal" color="grey-lighten-3" class="pa-3 rounded-lg" elevation="0">
              <div class="d-flex align-center mb-2">
                <v-icon size="18" color="success" class="me-2">mdi-play-circle</v-icon>
                <span class="text-body-2 font-weight-medium">{{ formatDate(item.start_date) }}</span>
              </div>
              <div class="d-flex align-center">
                <v-icon size="18" color="error" class="me-2">mdi-stop-circle</v-icon>
                <span class="text-body-2 font-weight-medium">{{ formatDate(item.end_date) }}</span>
              </div>
            </v-card>
          </template>

          <template v-slot:item.is_active="{ item }">
            <div class="d-flex align-center">
              <v-chip
                :color="item.is_active ? 'success' : 'grey'"
                size="small"
                class="me-3 font-weight-bold px-4"
                variant="flat"
              >
                <v-icon start size="16">
                  {{ item.is_active ? 'mdi-check-circle' : 'mdi-close-circle' }}
                </v-icon>
                {{ item.is_active ? 'ACTIVE' : 'INACTIVE' }}
              </v-chip>
              <v-switch
                v-model="item.is_active"
                :model-value="item.is_active === 1 || item.is_active === true"
                color="success"
                hide-details
                density="comfortable"
                inset
                @update:model-value="toggleStatus(item)"
              ></v-switch>
            </div>
          </template>

          <template v-slot:item.actions="{ item }">
            <div class="d-flex ga-2">
              <v-btn 
                icon="mdi-pencil" 
                size="small" 
                variant="flat" 
                color="warning" 
                class="action-btn"
                @click="openDialog(item)"
              ></v-btn>
              <v-btn 
                icon="mdi-delete" 
                size="small" 
                variant="flat" 
                color="error" 
                class="action-btn"
                @click="deletePeriod(item.id)"
              ></v-btn>
            </div>
          </template>

          <template v-slot:loading>
            <div class="pa-8">
              <v-progress-circular
                indeterminate
                color="primary"
                size="64"
              ></v-progress-circular>
            </div>
          </template>

          <template v-slot:no-data>
            <div class="pa-12 text-center">
              <v-avatar color="grey-lighten-3" size="80" class="mb-4">
                <v-icon size="48" color="grey-lighten-1">mdi-calendar-blank</v-icon>
              </v-avatar>
              <div class="text-h6 text-grey-darken-1 font-weight-medium">ไม่พบข้อมูลรอบการประเมิน</div>
              <div class="text-body-2 text-grey mt-2">เริ่มต้นโดยการสร้างรอบการประเมินใหม่</div>
            </div>
          </template>
        </v-data-table>
      </v-card>
    </v-container>

    <!-- Dialog -->
    <v-dialog v-model="dialog" max-width="600px" persistent transition="dialog-bottom-transition">
      <v-card rounded="xl" elevation="12">
        <v-card-title class="pa-0">
          <v-toolbar
            class="dialog-header"
            flat
          >
            <v-avatar :color="isEditing ? 'warning' : 'primary'" size="48" class="ms-4">
              <v-icon color="white" size="28">
                {{ isEditing ? 'mdi-pencil' : 'mdi-plus-circle' }}
              </v-icon>
            </v-avatar>
            <v-toolbar-title class="font-weight-bold text-h5 ms-3">
              {{ isEditing ? 'แก้ไขรายละเอียดรอบ' : 'เพิ่มรอบการประเมินใหม่' }}
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon="mdi-close" variant="text" size="large" @click="dialog = false"></v-btn>
          </v-toolbar>
        </v-card-title>

        <v-card-text class="pa-8">
          <v-form ref="form" v-model="valid">
            <v-row>
              <v-col cols="12" md="7">
                <div class="field-label mb-2">
                  <v-icon size="16" color="primary">mdi-tag</v-icon>
                  รหัสรอบประเมิน
                </div>
                <v-text-field
                  v-model="formModel.code"
                  placeholder="เช่น 1/2568"
                  prepend-inner-icon="mdi-tag-outline"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  hide-details="auto"
                  :rules="[v => !!v || 'จำเป็นต้องระบุรหัส']"
                ></v-text-field>
              </v-col>

              <v-col cols="12" md="5">
                <div class="field-label mb-2">
                  <v-icon size="16" color="primary">mdi-calendar</v-icon>
                  ปี พ.ศ.
                </div>
                <v-text-field
                  v-model="formModel.buddhist_year"
                  type="number"
                  prepend-inner-icon="mdi-calendar-range"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  hide-details="auto"
                  :rules="[v => !!v || 'ระบุปี']"
                ></v-text-field>
              </v-col>

              <v-col cols="12">
                <div class="field-label mb-2">
                  <v-icon size="16" color="primary">mdi-format-title</v-icon>
                  ชื่อรอบการประเมิน
                </div>
                <v-text-field
                  v-model="formModel.name_th"
                  placeholder="ระบุชื่อรอบการประเมิน"
                  prepend-inner-icon="mdi-text"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  hide-details="auto"
                  :rules="[v => !!v || 'กรุณากรอกชื่อรอบ']"
                ></v-text-field>
              </v-col>

              <v-col cols="12" md="6">
                <div class="field-label mb-2">
                  <v-icon size="16" color="success">mdi-calendar-start</v-icon>
                  วันที่เริ่มต้น
                </div>
                <v-text-field
                  v-model="formModel.start_date"
                  type="date"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  hide-details="auto"
                  persistent-placeholder
                  :rules="[v => !!v || 'เลือกวันที่']"
                ></v-text-field>
              </v-col>

              <v-col cols="12" md="6">
                <div class="field-label mb-2">
                  <v-icon size="16" color="error">mdi-calendar-end</v-icon>
                  วันที่สิ้นสุด
                </div>
                <v-text-field
                  v-model="formModel.end_date"
                  type="date"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  hide-details="auto"
                  persistent-placeholder
                  :rules="[v => !!v || 'เลือกวันที่']"
                ></v-text-field>
              </v-col>
              
              <v-col cols="12">
                <v-card variant="flat" color="success" class="rounded-lg pa-4" style="background: rgba(76, 175, 80, 0.1); border: 2px solid rgba(76, 175, 80, 0.3);">
                  <v-checkbox
                    v-model="formModel.is_active"
                    color="success"
                    hide-details
                    density="comfortable"
                  >
                    <template v-slot:label>
                      <div class="d-flex align-center">
                        <v-icon color="success" class="me-2">mdi-check-circle</v-icon>
                        <span class="font-weight-bold text-success">เปิดใช้งานทันทีหลังบันทึก</span>
                      </div>
                    </template>
                  </v-checkbox>
                </v-card>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions class="pa-6">
          <v-spacer></v-spacer>
          <v-btn 
            variant="outlined" 
            color="grey-darken-1" 
            size="large"
            rounded="lg"
            class="px-6 text-none font-weight-medium"
            @click="dialog = false"
          >
            ยกเลิก
          </v-btn>
          <v-btn 
            variant="flat" 
            :color="isEditing ? 'warning' : 'primary'" 
            size="large"
            rounded="lg"
            class="px-8 font-weight-bold text-none btn-hover"
            elevation="2"
            @click="savePeriod"
          >
            <v-icon start>{{ isEditing ? 'mdi-content-save' : 'mdi-plus-circle' }}</v-icon>
            บันทึกข้อมูล
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

// --- State Variables ---
const periods = ref([]);
const loading = ref(false);
const dialog = ref(false);
const valid = ref(false);
const search = ref('');

const formModel = ref({
  id: null,
  code: '',
  name_th: '',
  buddhist_year: new Date().getFullYear() + 543,
  start_date: '',
  end_date: '',
  is_active: false
});

// --- Table Configuration ---
const headers = [
  { title: 'รหัส', key: 'code', align: 'start', width: '140px' },
  { title: 'ปี พ.ศ.', key: 'buddhist_year', align: 'center', width: '110px' },
  { title: 'ชื่อรอบการประเมิน', key: 'name_th' },
  { title: 'ระยะเวลาประเมิน', key: 'period_date', sortable: false, width: '240px' },
  { title: 'สถานะการใช้งาน', key: 'is_active', align: 'start', width: '220px' },
  { title: 'จัดการ', key: 'actions', align: 'center', sortable: false, width: '120px' },
];

const isEditing = computed(() => !!formModel.value.id);

// --- API Methods (คงเดิม) ---
const getAuthHeader = () => {
  const token = localStorage.getItem('token');
  return { headers: { Authorization: `Bearer ${token}` } };
};

const fetchPeriods = async () => {
  loading.value = true;
  try {
    const res = await axios.get('http://localhost:7000/api/admin/periods', getAuthHeader());
    periods.value = res.data;
  } catch (error) {
    console.error("Error fetching periods:", error);
  } finally {
    loading.value = false;
  }
};

const savePeriod = async () => {
  if (!formModel.value.code || !formModel.value.name_th) return;

  try {
    const payload = { ...formModel.value };
    payload.is_active = payload.is_active ? 1 : 0; 

    if (isEditing.value) {
      await axios.put(`http://localhost:7000/api/admin/periods/${payload.id}`, payload, getAuthHeader());
    } else {
      await axios.post('http://localhost:7000/api/admin/periods', payload, getAuthHeader());
    }
    dialog.value = false;
    fetchPeriods();
  } catch (error) {
    alert('บันทึกไม่สำเร็จ: ' + (error.response?.data?.message || error.message));
  }
};

const deletePeriod = async (id) => {
  if (!confirm('ยืนยันการลบรอบการประเมินนี้?')) return;
  try {
    await axios.delete(`http://localhost:7000/api/admin/periods/${id}`, getAuthHeader());
    fetchPeriods();
  } catch (error) {
    alert('ลบไม่สำเร็จ: ' + error.message);
  }
};

const toggleStatus = async (item) => {
  try {
    const newStatus = item.is_active;
    await axios.put(`http://localhost:7000/api/admin/periods/${item.id}/status`, { 
      is_active: newStatus 
    }, getAuthHeader());
    fetchPeriods();
  } catch (error) {
    console.error("Toggle error:", error);
    item.is_active = !item.is_active;
  }
};

const openDialog = (item = null) => {
  if (item) {
    formModel.value = { 
      ...item,
      start_date: item.start_date ? item.start_date.split('T')[0] : '',
      end_date: item.end_date ? item.end_date.split('T')[0] : '',
      is_active: item.is_active === 1 || item.is_active === true
    };
  } else {
    formModel.value = {
      id: null,
      code: '',
      name_th: '',
      buddhist_year: new Date().getFullYear() + 543,
      start_date: '',
      end_date: '',
      is_active: false
    };
  }
  dialog.value = true;
};

const formatDate = (dateStr) => {
  if (!dateStr) return '-';
  const date = new Date(dateStr);
  return date.toLocaleDateString('th-TH', { day: '2-digit', month: 'short', year: 'numeric' });
};

onMounted(() => {
  fetchPeriods();
});

definePageMeta({
  middleware: 'auth'
})
</script>

<style scoped>
/* Background - เหมือนหน้า Dashboard */
.periods-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Header Gradient - เหมือนหน้า Dashboard */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

/* Dialog Header */
.dialog-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
}

.dialog-header :deep(.v-toolbar-title) {
  color: white;
}

.dialog-header :deep(.v-btn) {
  color: white;
}

/* Field Label Styling */
.field-label {
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  color: #667eea;
  display: flex;
  align-items: center;
  gap: 4px;
}

/* Search Field */
.search-field :deep(.v-field) {
  border: 2px solid rgba(102, 126, 234, 0.2);
  transition: all 0.3s ease;
}

.search-field :deep(.v-field--focused) {
  border-color: rgba(102, 126, 234, 0.5);
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

/* Button Hover Effect */
.btn-hover {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-hover:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(102, 126, 234, 0.35) !important;
}

/* Action Buttons */
.action-btn {
  transition: all 0.2s ease;
}

.action-btn:hover {
  transform: scale(1.1);
}

/* Table Row Hover */
:deep(.v-data-table__tr:hover) {
  background-color: rgba(102, 126, 234, 0.05) !important;
  transition: background-color 0.2s ease;
}

/* Custom Field Styling */
:deep(.v-field--variant-outlined) {
  border-radius: 12px !important;
}

:deep(.v-field--focused) {
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.15);
}

/* Chips */
:deep(.v-chip) {
  font-weight: 600;
}

/* Progress Circular */
:deep(.v-progress-circular) {
  margin: 0 auto;
}
</style>