<template>
  <v-app>
    <v-main class="auth-bg">
      <v-container class="fill-height" fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" sm="9" md="6" lg="4">
            
            <v-card class="mx-auto elevation-12 rounded-xl overflow-hidden" style="border: 1px solid rgba(102, 126, 234, 0.1);">
              
              <!-- Header Section with Gradient -->
              <v-sheet 
                class="header-gradient pa-10 text-center" 
                elevation="0"
              >
                <v-avatar color="white" size="80" class="mb-4 elevation-4">
                  <v-icon size="48" color="primary">
                    {{ isLogin ? 'mdi-shield-account' : 'mdi-account-plus' }}
                  </v-icon>
                </v-avatar>
                <h2 class="text-h4 font-weight-black text-white mb-2">
                  {{ isLogin ? 'ยินดีต้อนรับ' : 'สร้างบัญชีใหม่' }}
                </h2>
                <p class="text-body-1 text-white" style="opacity: 0.95">
                  {{ isLogin ? 'กรุณาเข้าสู่ระบบเพื่อใช้งาน VEC Skills' : 'กรอกข้อมูลให้ครบถ้วนเพื่อลงทะเบียน' }}
                </p>
              </v-sheet>

              <!-- Form Section -->
              <v-card-text class="pa-8">
                <v-window v-model="step">
                  
                  <!-- Login Form -->
                  <v-window-item :value="1">
                    <v-form ref="loginForm" @submit.prevent="handleLogin">
                      <div class="section-label mb-4">
                        <v-icon size="18" class="mr-1">mdi-login</v-icon>
                        ข้อมูลเข้าสู่ระบบ
                      </div>
                      
                      <v-text-field 
                        v-model="loginData.email" 
                        prepend-inner-icon="mdi-email-outline" 
                        label="อีเมล"
                        placeholder="example@email.com"
                        variant="outlined" 
                        class="mb-4 custom-field" 
                        color="primary"
                        hide-details="auto"
                        density="comfortable"
                      ></v-text-field>

                      <v-text-field 
                        v-model="loginData.password" 
                        prepend-inner-icon="mdi-lock-outline" 
                        label="รหัสผ่าน"
                        placeholder="กรอกรหัสผ่านของคุณ"
                        variant="outlined" 
                        :type="showPass ? 'text' : 'password'"
                        :append-inner-icon="showPass ? 'mdi-eye-off' : 'mdi-eye'"
                        @click:append-inner="showPass = !showPass" 
                        class="mb-6 custom-field" 
                        color="primary"
                        hide-details="auto"
                        density="comfortable"
                      ></v-text-field>

                      <v-btn 
                        block 
                        color="primary" 
                        size="x-large" 
                        class="rounded-lg font-weight-bold btn-hover mb-4" 
                        elevation="4"
                        type="submit" 
                        :loading="loading"
                      >
                        <v-icon start>mdi-login</v-icon>
                        เข้าสู่ระบบ
                      </v-btn>

                      <v-divider class="my-6">
                        <span class="text-caption text-grey px-3">หรือ</span>
                      </v-divider>

                      <div class="text-center">
                        <span class="text-body-2 text-grey-darken-2">ยังไม่มีบัญชีผู้ใช้งาน? </span>
                        <v-btn 
                          variant="text" 
                          color="primary" 
                          class="font-weight-bold px-2 text-none"
                          @click="step = 2"
                        >
                          ลงทะเบียนที่นี่
                          <v-icon end size="18">mdi-arrow-right</v-icon>
                        </v-btn>
                      </div>
                    </v-form>
                  </v-window-item>

                  <!-- Register Form -->
                  <v-window-item :value="2">
                    <v-form ref="registerForm" @submit.prevent="handleRegister">
                      
                      <div class="section-label mb-4">
                        <v-icon size="18" class="mr-1">mdi-account</v-icon>
                        ข้อมูลส่วนตัว
                      </div>
                      
                      <v-text-field 
                        v-model="regData.name_th" 
                        prepend-inner-icon="mdi-account-outline" 
                        label="ชื่อ-นามสกุล (ภาษาไทย)"
                        placeholder="กรอกชื่อ-นามสกุล"
                        variant="outlined" 
                        density="comfortable" 
                        class="mb-4 custom-field" 
                        color="primary"
                        hide-details="auto"
                      ></v-text-field>

                      <div class="section-label mb-4 mt-6">
                        <v-icon size="18" class="mr-1">mdi-office-building</v-icon>
                        ข้อมูลหน่วยงาน
                      </div>

                      <v-row dense>
                        <v-col cols="12" sm="6">
                          <v-select 
                            v-model="regData.department_id" 
                            :items="data_departments" 
                            item-title="name_th"
                            item-value="id" 
                            prepend-inner-icon="mdi-domain" 
                            label="แผนก" 
                            variant="outlined"
                            density="comfortable" 
                            color="primary" 
                            :loading="loadingData"
                            hide-details="auto"
                            class="mb-4 custom-field"
                          ></v-select>
                        </v-col>
                        <v-col cols="12" sm="6">
                          <v-select 
                            v-model="regData.org_group_id" 
                            :items="org_groups" 
                            item-title="name_th" 
                            item-value="id"
                            prepend-inner-icon="mdi-briefcase-outline" 
                            label="กลุ่มงาน" 
                            variant="outlined" 
                            density="comfortable"
                            color="primary" 
                            :loading="loadingData"
                            hide-details="auto"
                            class="mb-4 custom-field"
                          ></v-select>
                        </v-col>
                      </v-row>

                      <div class="section-label mb-4 mt-6">
                        <v-icon size="18" class="mr-1">mdi-shield-lock</v-icon>
                        ข้อมูลการเข้าสู่ระบบ
                      </div>

                      <v-text-field 
                        v-model="regData.email" 
                        prepend-inner-icon="mdi-email-outline" 
                        label="อีเมล"
                        placeholder="example@email.com"
                        variant="outlined" 
                        density="comfortable" 
                        class="mb-4 custom-field" 
                        color="primary"
                        hide-details="auto"
                      ></v-text-field>

                      <v-text-field 
                        v-model="regData.password" 
                        prepend-inner-icon="mdi-lock-outline" 
                        label="รหัสผ่าน"
                        placeholder="กรอกรหัสผ่าน (อย่างน้อย 6 ตัวอักษร)"
                        variant="outlined" 
                        density="comfortable" 
                        type="password" 
                        class="mb-6 custom-field"
                        color="primary"
                        hide-details="auto"
                      ></v-text-field>

                      <v-btn 
                        block 
                        color="primary" 
                        size="x-large" 
                        class="rounded-lg font-weight-bold btn-hover mb-4" 
                        elevation="4"
                        type="submit" 
                        :loading="loading"
                      >
                        <v-icon start>mdi-account-plus</v-icon>
                        สร้างบัญชีผู้ใช้
                      </v-btn>

                      <div class="text-center mt-4">
                        <v-btn 
                          variant="text" 
                          prepend-icon="mdi-arrow-left"
                          color="grey-darken-1"
                          class="text-none"
                          @click="step = 1"
                        >
                          กลับไปหน้าเข้าสู่ระบบ
                        </v-btn>
                      </div>
                    </v-form>
                  </v-window-item>

                </v-window>
              </v-card-text>
            </v-card>

            <!-- Footer -->
            <div class="text-center mt-8">
              <div class="text-body-2 text-grey-darken-1 mb-1">
                &copy; {{ new Date().getFullYear() }} <strong>VEC Skills</strong>
              </div>
              <div class="text-caption text-grey">
                <v-icon size="14" class="mr-1">mdi-lightning-bolt</v-icon>
                Powered by <span class="font-weight-bold">KruOak</span>
              </div>
            </div>

          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <!-- Snackbar -->
    <v-snackbar 
      v-model="snackbar.show" 
      :color="snackbar.color" 
      location="top right"
      elevation="12"
      class="mt-4"
      rounded="lg"
    >
      <div class="d-flex align-center">
        <v-icon start>
          {{ snackbar.color === 'success' ? 'mdi-check-circle' : 'mdi-alert-circle' }}
        </v-icon>
        <span class="font-weight-medium">{{ snackbar.text }}</span>
      </div>
      <template v-slot:actions>
        <v-btn icon="mdi-close" variant="text" size="small" @click="snackbar.show = false"></v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script setup>
import { ref, computed, reactive, onMounted } from 'vue' 
import axios from 'axios'
import { useAuthStore } from '~/stores/auth'

definePageMeta({
  layout: false
})

// --- State Variables (คงเดิม) ---
const step = ref(1)
const showPass = ref(false)
const loading = ref(false)
const loadingData = ref(false)
const isLogin = computed(() => step.value === 1)
const authStore = useAuthStore()
const router = useRouter()

const data_departments = ref([])
const org_groups = ref([])

const loginData = reactive({
  email: '',
  password: ''
})

const regData = reactive({
  email: '',
  password: '',
  name_th: '',
  department_id: null,
  org_group_id: null
})

const snackbar = reactive({
  show: false,
  text: '',
  color: 'success'
})

// --- Functions (คงเดิม) ---
const fetchDropdownData = async () => {
  loadingData.value = true
  try {
    const [deptRes, orgRes] = await Promise.all([
      axios.get('http://localhost:7000/api/users/departments_show_all'),
      axios.get('http://localhost:7000/api/users/org_groups_show_all')
    ])
    data_departments.value = deptRes.data.data_departments
    org_groups.value = orgRes.data.data_org_groups
  } catch (error) {
    console.error('Error:', error)
    showSnackbar('ไม่สามารถดึงข้อมูลแผนกหรือกลุ่มงานได้', 'error')
  } finally {
    loadingData.value = false
  }
}

onMounted(() => {
  fetchDropdownData()
})

const handleLogin = async () => {
  loading.value = true
  try {
    const response = await axios.post('http://localhost:7000/api/auth/login', {
      email: loginData.email,
      password: loginData.password
    })
    authStore.setAuth(response.data.token, response.data.user)
    showSnackbar('เข้าสู่ระบบสำเร็จ!', 'success')
    
    const role = authStore.user?.role
    if (role === 'admin') {
      await router.push('/')
    } else if (role === 'evaluatee') {
      await router.push('/EvidenceSubmission')  
    } else if (role === 'evaluator') {
      await router.push('/evaluation_index') 
    } else {
      await router.push('/')
    }
  } catch (error) {
    showSnackbar('Login Failed: ' + (error.response?.data?.message || error.message), 'error')
  } finally {
    loading.value = false
  }
}

const handleRegister = async () => {
  loading.value = true
  try {
    const payload = { ...regData }
    await axios.post('http://localhost:7000/api/users/create', payload)
    showSnackbar('สร้างบัญชีผู้ใช้งานสำเร็จ!', 'success')
    setTimeout(() => { step.value = 1 }, 1500)
  } catch (error) {
    showSnackbar('Registration Failed: ' + (error.response?.data?.message || error.message), 'error')
  } finally {
    loading.value = false
  }
}

const showSnackbar = (text, color) => {
  snackbar.text = text
  snackbar.color = color
  snackbar.show = true
}
</script>

<style scoped>
/* Background - เหมือนหน้า Dashboard */
.auth-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
  min-height: 100vh;
}

/* Header Gradient - เหมือนหน้า Dashboard */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.25);
}

/* Section Label Styling */
.section-label {
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  color: #667eea;
  display: flex;
  align-items: center;
}

/* Custom Field Styling */
.custom-field {
  transition: all 0.3s ease;
}

.custom-field:deep(.v-field) {
  border-radius: 10px;
}

.custom-field:deep(.v-field--focused) {
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.15);
}

/* Button Hover Effect */
.btn-hover {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-hover:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(102, 126, 234, 0.35) !important;
}

/* Card Styling */
.v-card {
  transition: all 0.3s ease;
  background: white;
}

/* Divider Styling */
.v-divider {
  border-color: rgba(0, 0, 0, 0.08);
}

/* Avatar Styling */
.v-avatar {
  border: 3px solid rgba(255, 255, 255, 0.3);
}

/* Snackbar Custom */
.v-snackbar:deep(.v-snackbar__wrapper) {
  border-radius: 12px;
}
</style>