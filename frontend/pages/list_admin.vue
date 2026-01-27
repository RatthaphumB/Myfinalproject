<template>
  <div class="pa-6 bg-grey-lighten-4" style="min-height: 100vh;">
    <v-row class="mb-4" align="center">
      <v-col cols="12" md="6">
        <h1 class="text-h5 font-weight-bold text-indigo-darken-2">
          <v-icon icon="mdi-account-cog" class="me-2"></v-icon>
          ระบบจัดการผู้ใช้งาน
        </h1>
        <div class="text-caption text-grey-darken-1">จัดการข้อมูลบัญชีผู้ใช้ สิทธิ์การเข้าถึง และสังกัดหน่วยงาน</div>
      </v-col>
      <v-col cols="12" md="6" class="d-flex justify-md-end">
        <v-btn
          color="indigo-darken-1"
          prepend-icon="mdi-account-plus"
          rounded="lg"
          elevation="2"
          class="text-none"
          @click="add"
        >
          เพิ่มผู้ใช้งานใหม่
        </v-btn>
      </v-col>
    </v-row>

    <v-card elevation="2" rounded="xl" class="overflow-hidden">
      <v-toolbar color="white" flat>
        <v-text-field
          v-model="search"
          prepend-inner-icon="mdi-magnify"
          label="ค้นหาด้วยชื่อ, อีเมล หรือแผนก..."
          variant="solo-filled"
          flat
          hide-details
          density="compact"
          class="ms-4"
          style="max-width: 400px;"
          rounded="lg"
        ></v-text-field>
        <v-spacer></v-spacer>
        <v-btn icon="mdi-refresh" variant="text" color="grey" @click="fetchData" :loading="loading"></v-btn>
      </v-toolbar>

      <v-divider></v-divider>

      <v-data-table
        :headers="headers"
        :items="users"
        :loading="loading"
        :search="search"
        hover
        class="pb-4"
      >
        <template v-slot:item.email="{ item }">
          <div class="d-flex align-center py-2">
            <v-avatar color="indigo-lighten-5" size="40" class="me-3 border">
              <span class="text-indigo-darken-1 font-weight-bold">{{ item.name_th?.charAt(0) || 'U' }}</span>
            </v-avatar>
            <div>
              <div class="font-weight-bold text-body-2">{{ item.name_th }}</div>
              <div class="text-caption text-grey">{{ item.email }}</div>
            </div>
          </div>
        </template>

        <template v-slot:item.role="{ value }">
          <v-chip
            :color="getRoleColor(value)"
            size="x-small"
            variant="flat"
            class="font-weight-bold text-uppercase"
            rounded="sm"
          >
            {{ value }}
          </v-chip>
        </template>

        <template v-slot:item.department_id="{ item }">
          <div class="text-body-2">{{ getDeptName(item.department_id) }}</div>
        </template>

        <template v-slot:item.status="{ value }">
          <v-chip
            :color="value === 'active' ? 'success' : 'error'"
            size="x-small"
            variant="tonal"
            class="font-weight-bold"
          >
            <v-icon start icon="mdi-circle" size="8"></v-icon>
            {{ value }}
          </v-chip>
        </template>

        <template v-slot:item.created_at="{ value }">
          <div class="text-caption text-grey">
            {{ new Date(value).toLocaleDateString('th-TH') }}
          </div>
        </template>

        <template v-slot:item.actions="{ item }">
          <div class="d-flex justify-end">
            <v-btn
              icon="mdi-pencil-outline"
              size="small"
              variant="text"
              color="amber-darken-3"
              @click="edit(item)"
            ></v-btn>
            <v-btn
              icon="mdi-trash-can-outline"
              size="small"
              variant="text"
              color="red-lighten-1"
              @click="remove(item.id)"
            ></v-btn>
          </div>
        </template>

        <template v-slot:loading>
          <v-skeleton-loader type="table-row@5"></v-skeleton-loader>
        </template>
      </v-data-table>
    </v-card>

    <v-dialog v-model="dialog" max-width="600" persistent transition="dialog-bottom-transition">
      <v-card rounded="xl">
        <v-toolbar :color="isEditing ? 'amber-darken-2' : 'indigo-darken-1'" flat>
          <v-icon class="ms-4" :icon="isEditing ? 'mdi-account-edit' : 'mdi-account-plus'"></v-icon>
          <v-toolbar-title class="font-weight-bold">
            {{ isEditing ? 'แก้ไขข้อมูลผู้ใช้' : 'สร้างบัญชีผู้ใช้ใหม่' }}
          </v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon="mdi-close" variant="text" @click="dialog = false"></v-btn>
        </v-toolbar>

        <v-card-text class="pa-6">
          <v-form>
            <h3 class="text-subtitle-2 font-weight-bold mb-4 text-indigo">ข้อมูลพื้นฐาน (Basic Info)</h3>
            <v-row dense>
              <v-col cols="12">
                <v-text-field
                  v-model="formModel.email"
                  label="อีเมล (Email Address)"
                  placeholder="example@domain.com"
                  prepend-inner-icon="mdi-email-outline"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="formModel.name_th"
                  label="ชื่อ-นามสกุล (ภาษาไทย)"
                  prepend-inner-icon="mdi-account-outline"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                ></v-text-field>
              </v-col>
            </v-row>

            <v-divider class="my-6"></v-divider>
            <h3 class="text-subtitle-2 font-weight-bold mb-4 text-indigo">สิทธิ์และสังกัด (Role & Affiliation)</h3>
            
            <v-row dense>
              <v-col cols="12" md="6">
                <v-select
                  v-model="formModel.role"
                  label="สิทธิ์การใช้งาน"
                  :items="['admin', 'evaluatee', 'evaluator']"
                  prepend-inner-icon="mdi-security"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                ></v-select>
              </v-col>
              <v-col cols="12" md="6">
                <v-select
                  v-model="formModel.status"
                  label="สถานะบัญชี"
                  :items="['active', 'disabled']"
                  prepend-inner-icon="mdi-check-circle-outline"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                ></v-select>
              </v-col>
              <v-col cols="12" md="6">
                <v-select
                  v-model="formModel.department_id"
                  label="แผนก (Department)"
                  :items="departmentList"
                  item-title="name_th"
                  item-value="id"
                  prepend-inner-icon="mdi-domain"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  clearable
                ></v-select>
              </v-col>
              <v-col cols="12" md="6">
                <v-select
                  v-model="formModel.org_group_id"
                  label="กลุ่มงาน (Org Group)"
                  :items="orgGroupList"
                  item-title="name_th"
                  item-value="id"
                  prepend-inner-icon="mdi-office-building-cog"
                  variant="outlined"
                  rounded="lg"
                  density="comfortable"
                  clearable
                ></v-select>
              </v-col>
            </v-row>

            <v-divider class="my-6"></v-divider>
            <h3 class="text-subtitle-2 font-weight-bold mb-4 text-indigo">ความปลอดภัย (Security)</h3>
            
            <v-text-field
              v-model="formModel.password_hash"
              label="รหัสผ่าน (Password)"
              :type="visible ? 'text' : 'password'"
              :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
              @click:append-inner="visible = !visible"
              prepend-inner-icon="mdi-lock-outline"
              variant="outlined"
              rounded="lg"
              density="comfortable"
              :hint="isEditing ? 'เว้นว่างไว้หากไม่ต้องการเปลี่ยน' : 'กำหนดรหัสผ่านเบื้องต้น'"
              persistent-hint
            ></v-text-field>
          </v-form>
        </v-card-text>

        <v-card-actions class="pa-6">
          <v-spacer></v-spacer>
          <v-btn variant="text" color="grey-darken-1" class="px-6" @click="dialog = false">ยกเลิก</v-btn>
          <v-btn
            :color="isEditing ? 'amber-darken-3' : 'indigo-darken-1'"
            variant="elevated"
            rounded="lg"
            class="px-8 font-weight-bold"
            @click="save"
          >
            {{ isEditing ? 'อัปเดตข้อมูล' : 'บันทึกข้อมูล' }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { onMounted, ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

const router = useRouter()
const users = ref([])
const loading = ref(false)
const dialog = ref(false)
const visible = ref(false)
const search = ref('') // สำหรับช่องค้นหา
const departmentList = ref([])
const orgGroupList = ref([])

const headers = [
  { title: 'ผู้ใช้งาน', key: 'email', align: 'start', width: '250px' },
  { title: 'สิทธิ์', key: 'role', align: 'center', width: '120px' },
  { title: 'แผนก/สังกัด', key: 'department_id', align: 'start' },
  { title: 'สถานะ', key: 'status', align: 'center', width: '120px' },
  { title: 'วันที่สร้าง', key: 'created_at', align: 'center', width: '150px' },
  { title: '', key: 'actions', align: 'end', sortable: false },
]

const formModel = ref({
  id: null,
  email: '',
  name_th: '',
  password_hash: '',
  status: 'active',
  role: 'evaluatee',
  department_id: null,
  org_group_id: null
})

const isEditing = computed(() => !!formModel.value.id)

// Helpers
const getRoleColor = (role) => {
  const colors = {
    admin: 'indigo-darken-2',
    evaluator: 'teal-darken-1',
    evaluatee: 'blue-grey-darken-1'
  }
  return colors[role] || 'grey'
}

const getDeptName = (id) => {
  const dept = departmentList.value.find(d => d.id === id)
  return dept ? dept.name_th : id || '-'
}

// Functions เหมือนเดิม...
const fetchData = async () => {
  loading.value = true
  try {
    const token = localStorage.getItem('token')
    const { data } = await axios.get("http://localhost:7000/api/admin/users", {
      headers: { Authorization: `Bearer ${token}` },
    })
    users.value = data
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

const fetchMasterData = async () => {
  try {
    const token = localStorage.getItem('token')
    const [deptRes, orgRes] = await Promise.all([
      axios.get("http://localhost:7000/api/admin/departments", { headers: { Authorization: `Bearer ${token}` } }),
      axios.get("http://localhost:7000/api/admin/org-groups", { headers: { Authorization: `Bearer ${token}` } })
    ])
    departmentList.value = deptRes.data
    orgGroupList.value = orgRes.data
  } catch (error) {
    console.error(error)
  }
}

const save = async () => {
  try {
    const token = localStorage.getItem('token')
    const headers = { Authorization: `Bearer ${token}` }
    const baseUrl = "http://localhost:7000/api/admin/users"
    if (isEditing.value) {
      await axios.put(`${baseUrl}/${formModel.value.id}`, formModel.value, { headers })
    } else {
      await axios.post(baseUrl, formModel.value, { headers })
    }
    dialog.value = false
    fetchData()
  } catch (error) {
    alert('Error: ' + error.message)
  }
}

const add = () => {
  formModel.value = { id: null, email: '', name_th: '', password_hash: '', status: 'active', role: 'evaluatee', department_id: null, org_group_id: null }
  dialog.value = true
}

const edit = (item) => {
  formModel.value = { ...item, password_hash: '' }
  dialog.value = true
}

const remove = async (id) => {
  if (!confirm("Confirm delete?")) return
  try {
    const token = localStorage.getItem('token')
    await axios.delete(`http://localhost:7000/api/admin/users/${id}`, { headers: { Authorization: `Bearer ${token}` } })
    fetchData()
  } catch (error) {
    alert(error.message)
  }
}

onMounted(() => {
  fetchData()
  fetchMasterData()
})

definePageMeta({ middleware: 'auth' })
</script>