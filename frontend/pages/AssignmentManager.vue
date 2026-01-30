<template>
  <div class="assignments-bg" style="min-height: 100vh;">
    <v-sheet class="header-gradient px-4 px-md-12 py-6 py-md-8 mb-n6" elevation="0">
      <v-row align="center" justify="space-between" no-gutters>
        <v-col cols="12" md="auto">
          <div class="d-flex align-center mb-2">
            <v-avatar color="white" size="48" class="mr-3 elevation-3 d-md-flex" :class="{'size-56': $vuetify.display.mdAndUp}">
              <v-icon color="primary" size="28" :size="$vuetify.display.mdAndUp ? 32 : 28">mdi-account-switch</v-icon>
            </v-avatar>
            <div>
              <h1 class="text-h5 text-md-h3 font-weight-black text-white mb-1">
                จัดการการจับคู่ประเมิน
              </h1>
              <div class="text-body-2 text-md-subtitle-1 text-white" style="opacity: 0.9">
                กำหนดคู่กรรมการและผู้รับการประเมินตามรอบปีที่ระบุ
              </div>
            </div>
          </div>
        </v-col>
      </v-row>
    </v-sheet>

    <v-container fluid class="px-4 px-md-12 pt-10 pt-md-12 pb-8">
      <v-row>
        <v-col cols="12" lg="4">
          <v-card 
            elevation="8" 
            rounded="xl" 
            class="h-100 form-card"
            :class="isEditing ? 'editing-mode' : 'create-mode'"
          >
            <v-card-item class="pa-4 pa-md-6 form-header">
              <template v-slot:prepend>
                <v-avatar 
                  :color="isEditing ? 'warning' : 'primary'" 
                  size="40"
                  class="d-md-flex"
                  :class="{'size-48': $vuetify.display.mdAndUp}"
                >
                  <v-icon color="white" size="24" :size="$vuetify.display.mdAndUp ? 28 : 24">
                    {{ isEditing ? 'mdi-pencil-circle' : 'mdi-plus-circle' }}
                  </v-icon>
                </v-avatar>
              </template>
              <v-card-title class="text-subtitle-1 text-md-h6 font-weight-bold">
                {{ isEditing ? 'แก้ไขข้อมูลการจับคู่' : 'สร้างการจับคู่ใหม่' }}
              </v-card-title>
              <v-card-subtitle v-if="isEditing" class="text-caption">
                กำลังแก้ไขรายการที่เลือก
              </v-card-subtitle>
            </v-card-item>

            <v-divider></v-divider>

            <v-card-text class="pa-4 pa-md-6">
              <div class="mb-6">
                <div class="field-label mb-3">
                  <v-icon size="18" color="primary">mdi-numeric-1-circle</v-icon>
                  เลือกรอบปีการประเมิน
                </div>
                <v-select
                  v-model="selectedPeriod"
                  :items="periods"
                  item-title="name_th"
                  item-value="id"
                  placeholder="เลือกรอบการประเมิน"
                  variant="outlined"
                  color="primary"
                  prepend-inner-icon="mdi-calendar-clock"
                  density="comfortable"
                  rounded="lg"
                  :loading="loading"
                  hide-details
                  class="custom-select"
                ></v-select>
              </div>

              <div class="mb-6">
                <div class="field-label mb-3">
                  <v-icon size="18" color="primary">mdi-numeric-2-circle</v-icon>
                  ผู้ประเมิน (กรรมการ)
                </div>
                <v-autocomplete
                  v-model="form.evaluator_id"
                  :items="evaluators"
                  item-title="name_th"
                  item-value="id"
                  placeholder="ค้นหาชื่อกรรมการ..."
                  variant="outlined"
                  color="primary"
                  prepend-inner-icon="mdi-account-tie"
                  density="comfortable"
                  rounded="lg"
                  clearable
                  :loading="loading"
                  hide-details
                  class="custom-select"
                >
                  <template v-slot:item="{ props, item }">
                    <v-list-item v-bind="props" :subtitle="item.raw.role">
                      <template v-slot:prepend>
                        <v-avatar color="primary" size="32" class="mr-2">
                          <v-icon color="white" size="16">mdi-account-tie</v-icon>
                        </v-avatar>
                      </template>
                    </v-list-item>
                  </template>
                </v-autocomplete>
              </div>

              <div class="mb-8">
                <div class="field-label mb-3">
                  <v-icon size="18" color="primary">mdi-numeric-3-circle</v-icon>
                  ผู้รับการประเมิน
                </div>
                <v-autocomplete
                  v-model="form.evaluatee_id"
                  :items="evaluatees"
                  item-title="name_th"
                  item-value="id"
                  placeholder="ค้นหาชื่อ หรือ แผนก..."
                  variant="outlined"
                  color="primary"
                  prepend-inner-icon="mdi-account-school"
                  density="comfortable"
                  rounded="lg"
                  clearable
                  :loading="loading"
                  :custom-filter="filterByDept"
                  hide-details
                  class="custom-select"
                >
                  <template v-slot:item="{ props, item }">
                    <v-list-item v-bind="props" :subtitle="item.raw.dept_name || 'ไม่ระบุแผนก'">
                      <template v-slot:prepend>
                        <v-avatar color="warning" size="32" class="mr-2">
                          <v-icon color="white" size="16">mdi-account-school</v-icon>
                        </v-avatar>
                      </template>
                    </v-list-item>
                  </template>
                </v-autocomplete>
              </div>

              <v-btn
                block
                :color="isEditing ? 'warning' : 'primary'"
                size="large"
                :size="$vuetify.display.mdAndUp ? 'x-large' : 'large'"
                rounded="lg"
                elevation="4"
                :prepend-icon="isEditing ? 'mdi-check-circle' : 'mdi-link-variant-plus'"
                :disabled="!isValidForm || loading"
                :loading="submitting"
                @click="saveAssignment"
                class="text-none font-weight-bold btn-hover mb-3"
              >
                {{ isEditing ? 'ยืนยันการแก้ไข' : 'ยืนยันการจับคู่' }}
              </v-btn>

              <v-btn
                v-if="isEditing"
                block
                variant="outlined"
                color="grey-darken-1"
                size="large"
                rounded="lg"
                class="text-none font-weight-medium"
                @click="cancelEdit"
              >
                <v-icon start>mdi-close-circle</v-icon>
                ยกเลิก
              </v-btn>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" lg="8">
          <v-card elevation="8" rounded="xl" class="overflow-hidden" style="border: 1px solid rgba(102, 126, 234, 0.1);">
            <v-toolbar flat color="white" class="px-4 py-2 py-md-0 h-auto" :height="$vuetify.display.mdAndUp ? 80 : 'auto'">
              <div class="d-flex flex-wrap align-center w-100 py-3 py-md-0">
                <div class="d-flex align-center flex-grow-1 mb-2 mb-md-0">
                  <v-avatar color="primary" size="32" class="mr-3 d-md-flex" :class="{'size-40': $vuetify.display.mdAndUp}">
                    <v-icon color="white" size="18" :size="$vuetify.display.mdAndUp ? 24 : 18">mdi-format-list-bulleted</v-icon>
                  </v-avatar>
                  <v-toolbar-title class="font-weight-bold text-subtitle-1 text-md-h6 mr-2">
                    ข้อมูลการจับคู่
                    <v-chip color="primary" size="x-small" :size="$vuetify.display.mdAndUp ? 'small' : 'x-small'" class="ml-1">
                      {{ filteredAssignments.length }}
                    </v-chip>
                  </v-toolbar-title>
                </div>
                
                <v-text-field
                  v-model="search"
                  prepend-inner-icon="mdi-magnify"
                  label="ค้นหาชื่อ..."
                  variant="outlined"
                  flat
                  hide-details
                  density="compact"
                  rounded="lg"
                  class="search-field w-100 w-md-auto"
                  style="min-width: 200px;"
                  :style="$vuetify.display.mdAndUp ? 'max-width: 320px;' : ''"
                ></v-text-field>
              </div>
            </v-toolbar>

            <v-divider></v-divider>

            <v-data-table
              :headers="headers"
              :items="filteredAssignments"
              :search="search"
              density="comfortable"
              hover
              class="assignments-table"
              :mobile-breakpoint="600"
            >
              <template v-slot:item.evaluator="{ item }">
                <div class="d-flex align-center py-3 py-md-4">
                  <v-avatar color="primary" size="40" class="mr-3 elevation-2" :class="{'size-48': $vuetify.display.mdAndUp}">
                    <v-icon color="white" size="20" :size="$vuetify.display.mdAndUp ? 24 : 20">mdi-account-tie</v-icon>
                  </v-avatar>
                  <div>
                    <div class="text-body-2 text-md-subtitle-1 font-weight-bold text-grey-darken-3 text-truncate" style="max-width: 140px; md:max-width: 100%;">
                      {{ item.evaluator_name }}
                    </div>
                    <div class="text-caption text-grey-darken-1">
                      <v-icon size="12" class="mr-1">mdi-shield-account</v-icon>
                      {{ item.evaluator_role }}
                    </div>
                  </div>
                </div>
              </template>

              <template v-slot:item.arrow>
                <div class="d-none d-md-flex justify-center">
                  <v-chip color="primary" variant="tonal" size="small" class="px-2">
                    <v-icon size="20">mdi-arrow-right-bold</v-icon>
                  </v-chip>
                </div>
              </template>

              <template v-slot:item.evaluatee="{ item }">
                <div class="d-flex align-center py-3 py-md-4">
                  <v-avatar color="warning" size="40" class="mr-3 elevation-2" :class="{'size-48': $vuetify.display.mdAndUp}">
                    <v-icon color="white" size="20" :size="$vuetify.display.mdAndUp ? 24 : 20">mdi-account-school</v-icon>
                  </v-avatar>
                  <div>
                    <div class="text-body-2 text-md-subtitle-1 font-weight-bold text-grey-darken-3 text-truncate" style="max-width: 140px; md:max-width: 100%;">
                      {{ item.evaluatee_name }}
                    </div>
                    <div class="text-caption text-grey-darken-1">
                      <v-icon size="12" class="mr-1">mdi-domain</v-icon>
                      {{ item.evaluatee_dept || 'ไม่ระบุแผนก' }}
                    </div>
                  </div>
                </div>
              </template>

              <template v-slot:item.actions="{ item }">
                <div class="d-flex justify-end ga-2">
                  <v-tooltip text="แก้ไข" location="top">
                    <template v-slot:activator="{ props }">
                      <v-btn
                        v-bind="props"
                        icon="mdi-pencil"
                        variant="flat"
                        color="warning"
                        size="small"
                        class="action-btn"
                        @click="editAssignment(item)"
                      ></v-btn>
                    </template>
                  </v-tooltip>

                  <v-tooltip text="ลบรายการ" location="top">
                    <template v-slot:activator="{ props }">
                      <v-btn
                        v-bind="props"
                        icon="mdi-delete"
                        variant="flat"
                        color="error"
                        size="small"
                        class="action-btn"
                        @click="removeAssignment(item.id)"
                      ></v-btn>
                    </template>
                  </v-tooltip>
                </div>
              </template>
              
              <template v-slot:no-data>
                <div class="pa-8 pa-md-12 text-center">
                  <v-avatar color="grey-lighten-3" size="60" class="mb-4" :class="{'size-80': $vuetify.display.mdAndUp}">
                    <v-icon size="36" :size="$vuetify.display.mdAndUp ? 48 : 36" color="grey-lighten-1">mdi-account-multiple-remove</v-icon>
                  </v-avatar>
                  <div class="text-subtitle-1 text-md-h6 text-grey-darken-1 font-weight-medium">
                    ยังไม่มีข้อมูลการจับคู่
                  </div>
                  <div class="text-caption text-md-body-2 text-grey mt-2">
                    เริ่มต้นโดยการสร้างการจับคู่ใหม่
                  </div>
                </div>
              </template>
            </v-data-table>
          </v-card>
        </v-col>
      </v-row>
    </v-container>

    <v-snackbar 
      v-model="snackbar" 
      :color="snackbarColor" 
      rounded="lg" 
      elevation="12"
      location="top right"
      class="mt-4"
    >
      <div class="d-flex align-center">
        <v-icon class="mr-2">
          {{ snackbarColor === 'success' ? 'mdi-check-circle' : 'mdi-alert-circle' }}
        </v-icon>
        <span class="font-weight-medium">{{ snackbarText }}</span>
      </div>
      <template v-slot:actions>
        <v-btn variant="text" size="small" icon="mdi-close" @click="snackbar = false"></v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

definePageMeta({
  middleware: 'auth'
})

const periods = ref([]);
const evaluators = ref([]);
const evaluatees = ref([]);
const assignments = ref([]);
const selectedPeriod = ref(null);
const form = ref({ evaluator_id: null, evaluatee_id: null });
const search = ref('');
const loading = ref(false);
const submitting = ref(false);
const isEditing = ref(false);
const editingId = ref(null);
const snackbar = ref(false);
const snackbarText = ref('');
const snackbarColor = ref('success');

const headers = [
  { title: 'ผู้ประเมิน (Evaluator)', key: 'evaluator', width: '40%' },
  { title: '', key: 'arrow', align: 'center', sortable: false, width: '80px' },
  { title: 'ผู้รับการประเมิน (Evaluatee)', key: 'evaluatee', width: '40%' },
  { title: 'จัดการ', key: 'actions', align: 'end', sortable: false, width: '120px' },
];

const filteredAssignments = computed(() => {
  if (!selectedPeriod.value) return [];
  return assignments.value.filter(a => a.period_id === selectedPeriod.value);
});

const isValidForm = computed(() => {
  return selectedPeriod.value && form.value.evaluator_id && form.value.evaluatee_id;
});

const filterByDept = (itemTitle, queryText, item) => {
    const textName = item.raw.name_th ? item.raw.name_th.toLowerCase() : '';
    const textDept = item.raw.dept_name ? item.raw.dept_name.toLowerCase() : '';
    const searchText = queryText.toLowerCase();
    return textName.indexOf(searchText) > -1 || textDept.indexOf(searchText) > -1;
};

const getAuthHeader = () => {
  return { headers: { Authorization: `Bearer ${localStorage.getItem('token')}` } };
};

const fetchData = async () => {
  loading.value = true;
  try {
    const header = getAuthHeader();
    const [resPeriods, resEvaluators, resEvaluatees, resAssignments] = await Promise.all([
        axios.get('http://localhost:7000/api/admin/periods', header),
        axios.get('http://localhost:7000/api/admin/users/evaluators', header),
        axios.get('http://localhost:7000/api/admin/users/evaluatees', header),
        axios.get('http://localhost:7000/api/admin/assignments', header)
    ]);
    periods.value = resPeriods.data;
    evaluators.value = resEvaluators.data;
    evaluatees.value = resEvaluatees.data;
    assignments.value = resAssignments.data;
    const activePeriod = periods.value.find(p => p.is_active === 1 || p.active === 1);
    if (activePeriod && !selectedPeriod.value) {
        selectedPeriod.value = activePeriod.id;
    }
  } catch (error) {
    showSnackbar('โหลดข้อมูลไม่สำเร็จ', 'error');
  } finally {
    loading.value = false;
  }
};

const saveAssignment = async () => {
  if (!isValidForm.value) return;
  submitting.value = true;
  try {
    const payload = {
        period_id: selectedPeriod.value,
        evaluator_id: form.value.evaluator_id,
        evaluatee_id: form.value.evaluatee_id
    };
    if (isEditing.value) {
        await axios.put(`http://localhost:7000/api/admin/assignments/${editingId.value}`, payload, getAuthHeader());
        showSnackbar('แก้ไขข้อมูลเรียบร้อย');
    } else {
        await axios.post('http://localhost:7000/api/admin/assignments', payload, getAuthHeader());
        showSnackbar('จับคู่สำเร็จเรียบร้อย');
    }
    const res = await axios.get('http://localhost:7000/api/admin/assignments', getAuthHeader());
    assignments.value = res.data;
    cancelEdit(); 
  } catch (error) {
    showSnackbar(error.response?.data?.message || 'เกิดข้อผิดพลาด', 'error');
  } finally {
    submitting.value = false;
  }
};

const editAssignment = (item) => {
    isEditing.value = true;
    editingId.value = item.id;
    form.value = {
        evaluator_id: item.evaluator_id,
        evaluatee_id: item.evaluatee_id
    };
    selectedPeriod.value = item.period_id;
};

const cancelEdit = () => {
    isEditing.value = false;
    editingId.value = null;
    form.value = { evaluator_id: null, evaluatee_id: null };
};

const removeAssignment = async (id) => {
  if (!confirm('ยืนยันที่จะลบการจับคู่นี้?')) return;
  try {
    await axios.delete(`http://localhost:7000/api/admin/assignments/${id}`, getAuthHeader());
    assignments.value = assignments.value.filter(a => a.id !== id);
    showSnackbar('ลบรายการเรียบร้อย');
    if (editingId.value === id) cancelEdit();
  } catch (error) {
    showSnackbar('ลบไม่สำเร็จ', 'error');
  }
};

const showSnackbar = (text, color = 'success') => {
  snackbarText.value = text;
  snackbarColor.value = color;
  snackbar.value = true;
};

onMounted(() => {
  fetchData();
});
</script>

<style scoped>
/* Background - เหมือนหน้า Dashboard */
.assignments-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Header Gradient - เหมือนหน้า Dashboard */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

/* Form Card Styling */
.form-card {
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.form-card.create-mode {
  border-color: rgba(102, 126, 234, 0.3);
}

.form-card.editing-mode {
  border-color: rgba(255, 152, 0, 0.4);
  box-shadow: 0 8px 32px rgba(255, 152, 0, 0.2) !important;
}

.form-header {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.05) 0%, rgba(118, 75, 162, 0.05) 100%);
}

/* Field Label Styling */
.field-label {
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  color: #667eea;
  display: flex;
  align-items: center;
  gap: 6px;
}

/* Custom Select Styling */
.custom-select :deep(.v-field) {
  transition: all 0.3s ease;
  border: 2px solid rgba(102, 126, 234, 0.1);
}

.custom-select :deep(.v-field--focused) {
  border-color: rgba(102, 126, 234, 0.4);
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
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
  transform: scale(1.15);
}

/* Table Styling */
.assignments-table :deep(.v-data-table__tr:hover) {
  background-color: rgba(102, 126, 234, 0.05) !important;
  transition: background-color 0.2s ease;
}

/* Avatar Styling */
.v-avatar {
  border: 2px solid rgba(255, 255, 255, 0.5);
}

/* Custom Scrollbar */
:deep(.v-data-table__wrapper) {
  scrollbar-width: thin;
  scrollbar-color: rgba(102, 126, 234, 0.3) transparent;
}

:deep(.v-data-table__wrapper)::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

:deep(.v-data-table__wrapper)::-webkit-scrollbar-track {
  background: transparent;
}

:deep(.v-data-table__wrapper)::-webkit-scrollbar-thumb {
  background-color: rgba(102, 126, 234, 0.3);
  border-radius: 4px;
}

:deep(.v-data-table__wrapper)::-webkit-scrollbar-thumb:hover {
  background-color: rgba(102, 126, 234, 0.5);
}

/* Helper utility for specific sizes if needed via CSS */
.size-40 { width: 40px !important; height: 40px !important; }
.size-48 { width: 48px !important; height: 48px !important; }
.size-56 { width: 56px !important; height: 56px !important; }
.size-80 { width: 80px !important; height: 80px !important; }
</style>