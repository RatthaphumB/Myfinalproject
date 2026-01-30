<template>
  <div class="periods-bg" style="min-height: 100vh;">
    <v-sheet class="header-gradient px-4 px-md-12 py-6 py-md-8 mb-n6 rounded-b-xl" elevation="0">
      <v-row align="center" justify="space-between" no-gutters>
        <v-col cols="12" md="auto" class="mb-4 mb-md-0 text-center text-md-left">
          <div class="d-flex flex-column flex-md-row align-center">
            <v-avatar color="white" size="56" class="mb-3 mb-md-0 mr-md-4 elevation-3 border-opacity-25">
              <v-icon color="primary" size="32">mdi-calendar-clock</v-icon>
            </v-avatar>
            
            <div>
              <h1 class="text-h5 text-md-h3 font-weight-black text-white mb-1">
                ระบบจัดการรอบการประเมิน
              </h1>
              <div class="text-body-2 text-subtitle-1 text-white opacity-90">
                จัดการช่วงเวลาการส่งหลักฐานและเกณฑ์การให้คะแนน
              </div>
            </div>
          </div>
        </v-col>
      </v-row>
    </v-sheet>

    <v-container fluid class="px-4 px-md-12 pt-10 pt-md-12 pb-8 position-relative" style="z-index: 2;">
      <v-card elevation="8" rounded="xl" class="overflow-hidden border-thin">
        
        <div class="pa-4 pa-md-5 bg-white">
          <v-row align="center" dense>
            <v-col cols="12" md="5">
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
              ></v-text-field>
            </v-col>
            
            <v-spacer class="d-none d-md-block"></v-spacer>
            
            <v-col cols="12" md="auto">
              <v-btn 
                color="primary" 
                variant="flat" 
                prepend-icon="mdi-plus-circle" 
                rounded="lg"
                size="large"
                class="text-none font-weight-bold btn-hover"
                elevation="2"
                block
                @click="openDialog()"
              >
                เปิดรอบประเมินใหม่
              </v-btn>
            </v-col>
          </v-row>
        </div>

        <v-divider></v-divider>

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
              size="small"
              class="font-weight-bold"
            >
              <v-icon start size="14">mdi-tag</v-icon>
              {{ item.code }}
            </v-chip>
          </template>

          <template v-slot:item.buddhist_year="{ item }">
            <span class="text-body-1 font-weight-bold text-primary">{{ item.buddhist_year }}</span>
          </template>

          <template v-slot:item.name_th="{ item }">
            <div class="py-2">
              <div class="text-body-2 text-md-subtitle-1 font-weight-bold text-grey-darken-3">
                {{ item.name_th }}
              </div>
            </div>
          </template>

          <template v-slot:item.period_date="{ item }">
            <v-card variant="tonal" color="grey-lighten-4" class="pa-2 pa-md-3 rounded-lg border-0" elevation="0">
              <div class="d-flex align-center mb-1">
                <v-icon size="16" color="success" class="me-2">mdi-play-circle</v-icon>
                <span class="text-caption text-md-body-2 font-weight-medium">{{ formatDate(item.start_date) }}</span>
              </div>
              <div class="d-flex align-center">
                <v-icon size="16" color="error" class="me-2">mdi-stop-circle</v-icon>
                <span class="text-caption text-md-body-2 font-weight-medium">{{ formatDate(item.end_date) }}</span>
              </div>
            </v-card>
          </template>

          <template v-slot:item.is_active="{ item }">
            <div class="d-flex align-center justify-space-between justify-md-start" style="min-width: 160px;">
              <v-chip
                :color="item.is_active ? 'success' : 'grey'"
                size="small"
                class="me-3 font-weight-bold"
                variant="flat"
              >
                {{ item.is_active ? 'ACTIVE' : 'INACTIVE' }}
              </v-chip>
              <v-switch
                v-model="item.is_active"
                :model-value="item.is_active === 1 || item.is_active === true"
                color="success"
                hide-details
                density="compact"
                inset
                @click.stop
                @update:model-value="toggleStatus(item)"
              ></v-switch>
            </div>
          </template>

          <template v-slot:item.actions="{ item }">
            <div class="d-flex ga-2 justify-end justify-md-center">
              <v-btn 
                icon="mdi-pencil" 
                size="small" 
                variant="tonal" 
                color="warning" 
                @click="openDialog(item)"
              ></v-btn>
              <v-btn 
                icon="mdi-delete" 
                size="small" 
                variant="tonal" 
                color="error" 
                @click="deletePeriod(item.id)"
              ></v-btn>
            </div>
          </template>

          <template v-slot:no-data>
            <div class="pa-8 text-center">
              <v-icon size="48" color="grey-lighten-1" class="mb-2">mdi-calendar-blank</v-icon>
              <div class="text-body-1 text-grey-darken-1">ไม่พบข้อมูล</div>
            </div>
          </template>
        </v-data-table>
      </v-card>
    </v-container>

    <v-dialog 
      v-model="dialog" 
      :fullscreen="$vuetify.display.smAndDown"
      :transition="$vuetify.display.smAndDown ? 'dialog-bottom-transition' : 'dialog-bottom-transition'"
      max-width="600px" 
      persistent 
      scrollable
    >
      <v-card rounded="xl" elevation="12">
        <v-card-title class="pa-0">
          <v-toolbar class="dialog-header" flat density="comfortable">
            <v-btn icon="mdi-close" variant="text" @click="dialog = false" class="d-md-none"></v-btn>
            <v-toolbar-title class="font-weight-bold text-h6 text-md-h5 ms-2">
               {{ isEditing ? 'แก้ไขรายละเอียด' : 'เพิ่มรอบการประเมิน' }}
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon="mdi-close" variant="text" @click="dialog = false" class="d-none d-md-flex"></v-btn>
            
            <v-btn 
              variant="text" 
              class="d-md-none font-weight-bold" 
              @click="savePeriod"
            >
              บันทึก
            </v-btn>
          </v-toolbar>
        </v-card-title>

        <v-card-text class="pa-4 pa-md-8 bg-grey-lighten-5">
          <v-form ref="form" v-model="valid">
            <v-card class="pa-4 mb-4 rounded-lg border-0" elevation="0" color="white">
              <div class="text-subtitle-2 font-weight-bold text-primary mb-4">ข้อมูลทั่วไป</div>
              <v-row dense>
                <v-col cols="12" sm="7">
                  <div class="field-label mb-1">รหัสรอบประเมิน</div>
                  <v-text-field
                    v-model="formModel.code"
                    placeholder="เช่น 1/2568"
                    prepend-inner-icon="mdi-tag-outline"
                    variant="outlined"
                    rounded="lg"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-2"
                    :rules="[v => !!v || 'จำเป็น']"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="5">
                  <div class="field-label mb-1">ปี พ.ศ.</div>
                  <v-text-field
                    v-model="formModel.buddhist_year"
                    type="number"
                    prepend-inner-icon="mdi-calendar-range"
                    variant="outlined"
                    rounded="lg"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-2"
                    :rules="[v => !!v || 'จำเป็น']"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <div class="field-label mb-1">ชื่อรอบการประเมิน</div>
                  <v-text-field
                    v-model="formModel.name_th"
                    placeholder="ระบุชื่อรอบ"
                    prepend-inner-icon="mdi-text"
                    variant="outlined"
                    rounded="lg"
                    density="comfortable"
                    hide-details="auto"
                    :rules="[v => !!v || 'จำเป็น']"
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-card>

            <v-card class="pa-4 rounded-lg border-0" elevation="0" color="white">
              <div class="text-subtitle-2 font-weight-bold text-primary mb-4">ระยะเวลาและสถานะ</div>
              <v-row dense>
                <v-col cols="12" sm="6">
                  <div class="field-label mb-1 text-success">วันที่เริ่มต้น</div>
                  <v-text-field
                    v-model="formModel.start_date"
                    type="date"
                    variant="outlined"
                    rounded="lg"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-2"
                    :rules="[v => !!v || 'จำเป็น']"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <div class="field-label mb-1 text-error">วันที่สิ้นสุด</div>
                  <v-text-field
                    v-model="formModel.end_date"
                    type="date"
                    variant="outlined"
                    rounded="lg"
                    density="comfortable"
                    hide-details="auto"
                    class="mb-2"
                    :rules="[v => !!v || 'จำเป็น']"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-list-item class="px-0 mt-2">
                    <template v-slot:prepend>
                      <v-checkbox
                        v-model="formModel.is_active"
                        color="success"
                        hide-details
                        density="compact"
                      ></v-checkbox>
                    </template>
                    <v-list-item-title class="font-weight-bold text-success">เปิดใช้งานทันที</v-list-item-title>
                    <v-list-item-subtitle>ผู้ใช้จะเห็นรอบการประเมินนี้เมื่อบันทึก</v-list-item-subtitle>
                  </v-list-item>
                </v-col>
              </v-row>
            </v-card>
          </v-form>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions class="pa-4 d-none d-md-flex bg-white">
          <v-spacer></v-spacer>
          <v-btn 
            variant="outlined" 
            color="grey-darken-1" 
            size="large"
            rounded="lg"
            class="px-6"
            @click="dialog = false"
          >
            ยกเลิก
          </v-btn>
          <v-btn 
            variant="flat" 
            :color="isEditing ? 'warning' : 'primary'" 
            size="large"
            rounded="lg"
            class="px-8 font-weight-bold"
            elevation="2"
            @click="savePeriod"
          >
            <v-icon start>{{ isEditing ? 'mdi-content-save' : 'mdi-plus-circle' }}</v-icon>
            บันทึก
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useDisplay } from 'vuetify'; // เพิ่ม useDisplay เพื่อเช็คขนาดหน้าจอ

// --- Display Logic ---
const { mobile } = useDisplay(); // ใช้ตัวแปร mobile เพื่อปรับ Layout ตามเงื่อนไข

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
  { title: 'รหัส', key: 'code', align: 'start', width: '100px' },
  { title: 'ปี', key: 'buddhist_year', align: 'center', width: '80px' },
  { title: 'ชื่อรอบการประเมิน', key: 'name_th' },
  { title: 'ระยะเวลาประเมิน', key: 'period_date', sortable: false, width: '220px' },
  { title: 'สถานะ', key: 'is_active', align: 'start', width: '150px' },
  { title: 'จัดการ', key: 'actions', align: 'end', sortable: false, width: '100px' },
];

const isEditing = computed(() => !!formModel.value.id);

// --- API Methods (Mock Example) ---
// ในการใช้งานจริง เปลี่ยน URL เป็น endpoint ของคุณ
const getAuthHeader = () => {
  const token = localStorage.getItem('token');
  return { headers: { Authorization: `Bearer ${token}` } };
};

const fetchPeriods = async () => {
  loading.value = true;
  // จำลองข้อมูล (Mock Data) ถ้า API ยังไม่พร้อม
  setTimeout(() => {
    periods.value = [
       { id: 1, code: '1/2568', buddhist_year: 2568, name_th: 'ประเมินรอบที่ 1 ประจำปี 2568', start_date: '2025-01-01', end_date: '2025-06-30', is_active: true },
       { id: 2, code: '2/2567', buddhist_year: 2567, name_th: 'ประเมินรอบที่ 2 ประจำปี 2567', start_date: '2024-07-01', end_date: '2024-12-31', is_active: false },
    ];
    loading.value = false;
  }, 500);
  
  // try {
  //   const res = await axios.get('http://localhost:7000/api/admin/periods', getAuthHeader());
  //   periods.value = res.data;
  // } catch (error) {
  //   console.error("Error fetching periods:", error);
  // } finally {
  //   loading.value = false;
  // }
};

const savePeriod = async () => {
  // Validation แบบง่าย
  if (!formModel.value.code || !formModel.value.name_th) return;
  alert('บันทึกข้อมูลเรียบร้อย (Mock Action)');
  dialog.value = false;
  
  // Logic เดิม
  /*
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
  */
};

const deletePeriod = async (id) => {
  if (!confirm('ยืนยันการลบรอบการประเมินนี้?')) return;
  alert('ลบข้อมูลเรียบร้อย (Mock Action)');
  // logic เดิม...
};

const toggleStatus = async (item) => {
  // logic เดิม...
  console.log('Toggle status:', item.id);
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
/* Background */
.periods-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Header Gradient */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

/* Dialog Header */
.dialog-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.dialog-header :deep(.v-toolbar-title) {
  color: white;
}

.dialog-header :deep(.v-btn) {
  color: white;
}

/* Field Label */
.field-label {
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  color: #5c6bc0;
}

/* Search Field */
.search-field :deep(.v-field) {
  border: 1px solid rgba(102, 126, 234, 0.15);
  transition: all 0.3s ease;
}

.search-field :deep(.v-field--focused) {
  border-color: rgba(102, 126, 234, 0.5);
}

/* Button Hover */
.btn-hover {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-hover:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(102, 126, 234, 0.35) !important;
}

/* Border Utility */
.border-thin {
  border: 1px solid rgba(0, 0, 0, 0.08);
}

/* Text Opacity */
.opacity-90 {
  opacity: 0.9;
}
</style>