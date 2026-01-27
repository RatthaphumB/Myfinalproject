<template>
  <v-container fluid class="bg-grey-lighten-4 min-h-screen">
    <v-card class="mb-4 elevation-2" rounded="lg">
       <div class="d-flex align-center pa-4 bg-indigo-darken-2">
          <v-avatar color="white" class="mr-3" size="48">
             <v-icon color="indigo-darken-2" icon="mdi-shape-plus" size="large"></v-icon>
          </v-avatar>
          <div>
            <div class="text-h6 font-weight-bold">ระบบจัดการหมวดหมู่วิชาชีพ</div>
            <div class="text-caption text-indigo-lighten-4">Vocational Categories Management</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn color="error" variant="flat" prepend-icon="mdi-logout" @click="logout" size="small" class="text-capitalize">
            ออกจากระบบ
          </v-btn>
       </div>
    </v-card>

    <v-card elevation="2" rounded="lg">
      <v-data-table
        :headers="headers"
        :items="categories"
        :search="search"
        :loading="loading"
        hover
        class="rounded-lg"
      >
        <template v-slot:top>
          <v-toolbar flat color="white" class="px-2 pt-2">
             <v-text-field
                v-model="search"
                density="compact"
                variant="outlined"
                label="ค้นหาชื่อ หรือ รหัส..."
                prepend-inner-icon="mdi-magnify"
                hide-details
                single-line
                style="max-width: 350px;"
                class="mr-4 ml-2"
                rounded="lg"
                color="indigo"
                bg-color="grey-lighten-5"
             ></v-text-field>
             
             <v-spacer></v-spacer>
             
             <v-btn
                color="indigo-darken-1"
                variant="flat"
                prepend-icon="mdi-plus-circle"
                rounded="lg"
                height="40"
                class="mr-2 text-capitalize"
                @click="add"
             >
                เพิ่มหมวดหมู่ใหม่
             </v-btn>
          </v-toolbar>
        </template>

        <template v-slot:item.code="{ value }">
           <v-chip color="indigo" variant="tonal" label size="small" class="font-weight-bold">
             <v-icon start icon="mdi-barcode" size="x-small"></v-icon>
             {{ value }}
           </v-chip>
        </template>

        <template v-slot:item.created_at="{ value }">
           <span class="text-caption text-grey-darken-1">
             {{ value ? new Date(value).toLocaleDateString('th-TH', { day: '2-digit', month: 'short', year: '2-digit', hour: '2-digit', minute: '2-digit' }) : '-' }}
           </span>
        </template>
        
        <template v-slot:item.actions="{ item }">
          <div class="d-flex justify-end">
             <v-tooltip text="แก้ไข" location="top">
                <template v-slot:activator="{ props }">
                    <v-btn v-bind="props" icon="mdi-pencil-outline" variant="text" color="amber-darken-3" density="comfortable" @click="edit(item)"></v-btn>
                </template>
             </v-tooltip>
             
             <v-tooltip text="ลบข้อมูล" location="top">
                <template v-slot:activator="{ props }">
                    <v-btn v-bind="props" icon="mdi-trash-can-outline" variant="text" color="red-lighten-1" density="comfortable" @click="remove(item.id)"></v-btn>
                </template>
             </v-tooltip>
          </div>
        </template>

        <template v-slot:no-data>
           <div class="pa-8 text-center text-grey">
             <v-icon size="64" class="mb-2 text-grey-lighten-2">mdi-database-search-outline</v-icon>
             <div class="text-h6 text-grey-darken-1">ไม่พบข้อมูล</div>
             <v-btn color="indigo" variant="outlined" rounded="lg" prepend-icon="mdi-refresh" @click="fetchData" class="mt-4">โหลดข้อมูลใหม่</v-btn>
           </div>
        </template>
      </v-data-table>
    </v-card>

    <v-dialog v-model="dialog" max-width="500px" transition="dialog-top-transition" persistent>
      <v-form ref="formRef" @submit.prevent="save">
        <v-card rounded="xl" elevation="10">
          <v-toolbar :color="isEditing ? 'amber-darken-1' : 'indigo-darken-1'" class="px-2">
             <v-toolbar-title class="text-subtitle-1 font-weight-bold ml-2 text-white">
               <v-icon :icon="isEditing ? 'mdi-pencil-box-multiple' : 'mdi-plus-box-multiple'" start></v-icon>
               {{ isEditing ? 'แก้ไขข้อมูลหมวดหมู่' : 'เพิ่มหมวดหมู่ใหม่' }}
             </v-toolbar-title>
             <v-btn icon="mdi-close" variant="text" color="white" @click="dialog = false"></v-btn>
          </v-toolbar>

          <v-card-text class="pa-6">
             <v-row dense>
               <v-col cols="12">
                 <div class="text-subtitle-2 font-weight-bold mb-1 text-grey-darken-2">รหัสหมวดหมู่ (Required)</div>
                 <v-text-field
                   v-model="formModel.code"
                   :rules="[v => !!v || 'กรุณาระบุรหัส']"
                   placeholder="ตัวอย่าง: CAT01"
                   variant="outlined"
                   color="indigo"
                   density="comfortable"
                   prepend-inner-icon="mdi-barcode-scan"
                   rounded="lg"
                 ></v-text-field>
               </v-col>
               
               <v-col cols="12">
                 <div class="text-subtitle-2 font-weight-bold mb-1 text-grey-darken-2">ชื่อหมวดหมู่ภาษาไทย (Required)</div>
                 <v-text-field
                   v-model="formModel.name_th"
                   :rules="[v => !!v || 'กรุณาระบุชื่อไทย']"
                   placeholder="ตัวอย่าง: อุตสาหกรรม"
                   variant="outlined"
                   color="indigo"
                   density="comfortable"
                   prepend-inner-icon="mdi-format-list-text"
                   rounded="lg"
                 ></v-text-field>
               </v-col>
             </v-row>
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions class="pa-4 bg-grey-lighten-5">
            <v-spacer></v-spacer>
            <v-btn color="grey-darken-1" variant="text" class="px-4" rounded="lg" @click="dialog = false">ยกเลิก</v-btn>
            <v-btn
              :color="isEditing ? 'amber-darken-2' : 'indigo-darken-1'"
              variant="flat"
              width="140"
              rounded="lg"
              prepend-icon="mdi-content-save-check"
              type="submit"
              :loading="loading"
            >
              {{ isEditing ? 'อัปเดตข้อมูล' : 'บันทึกข้อมูล' }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
    </v-dialog>

    <v-snackbar v-model="snackbar.show" :color="snackbar.color" timeout="3000" rounded="lg">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="snackbar.show = false">ปิด</v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script setup>
import { onMounted, ref, computed } from 'vue'
import { useRouter } from 'vue-router' 
import axios from 'axios'

const router = useRouter()
const search = ref('')
const loading = ref(false)
const dialog = ref(false)
const formRef = ref(null)

const snackbar = ref({
  show: false,
  text: '',
  color: 'success'
})

const createNewRecord = () => ({
  code: '',
  name_th: '',
})

const formModel = ref(createNewRecord())
const categories = ref([]); 

const isEditing = computed(() => !!formModel.value.id)

const headers = [
  { title: 'ID', key: 'id', align: 'start', width: '80px' },
  { title: 'รหัส (CODE)', key: 'code', align: 'start', width: '150px' },
  { title: 'ชื่อหมวดหมู่', key: 'name_th', align: 'start' },
  { title: 'วันที่สร้าง', key: 'created_at', align: 'end', width: '180px' },
  { title: 'จัดการ', key: 'actions', align: 'end', sortable: false, width: '120px' },
]

// API Configuration
const getHeader = () => ({
  headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
})

const showMessage = (text, color = 'success') => {
  snackbar.value = { show: true, text, color }
}

const fetchData = async () => {
  loading.value = true
  try {
    // แก้ไข URL ให้ตรงกับ Resource ที่คุณต้องการจัดการ
    const { data } = await axios.get("http://localhost:7000/api/cat/list_all", getHeader());
    categories.value = data.list || []; 
  } catch (error) {
    showMessage('ไม่สามารถโหลดข้อมูลได้', 'error')
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  if (!localStorage.getItem('token')) {
    router.push('/login')
    return
  }
  fetchData(); 
})

function add() {
  formModel.value = createNewRecord()
  dialog.value = true
}

function edit(item) {
  formModel.value = { ...item } // ใช้วิธี Spread เพื่อป้องกัน Reference ชนกับตาราง
  dialog.value = true
}

async function remove(id) {
  if (!confirm("คุณต้องการลบข้อมูลนี้ใช่หรือไม่?")) return
  
  loading.value = true
  try {
    await axios.delete(`http://localhost:7000/api/cat/catdelete/${id}`, getHeader())
    showMessage('ลบข้อมูลเรียบร้อยแล้ว')
    await fetchData(); 
  } catch (error) {
    showMessage('ลบข้อมูลไม่สำเร็จ', 'error')
  } finally {
    loading.value = false
  }
}

async function save() {
  const { valid } = await formRef.value.validate()
  if (!valid) return

  loading.value = true
  try {
    if (isEditing.value) {
        await axios.put("http://localhost:7000/api/cat/updateCatigory", formModel.value, getHeader())
        showMessage('อัปเดตข้อมูลสำเร็จ')
    } else {
        await axios.post("http://localhost:7000/api/cat/createCatigory", formModel.value, getHeader())
        showMessage('บันทึกข้อมูลใหม่สำเร็จ')
    }
    await fetchData();
    dialog.value = false
  } catch (error) {
    showMessage(error.response?.data?.message || 'เกิดข้อผิดพลาดในการบันทึก', 'error')
  } finally {
    loading.value = false
  }
}

const logout = () => {
  localStorage.removeItem('token')
  router.push('/login')
}
</script>

<style scoped>
.min-h-screen {
  min-height: 100vh;
}
</style>