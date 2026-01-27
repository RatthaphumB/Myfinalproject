<script setup>
import { ref, computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useAuthStore } from '~/stores/auth'

const route = useRoute()
const router = useRouter()
const config = useRuntimeConfig()
const authStore = useAuthStore()

const assignmentId = route.query.id 
const evidenceDialog = ref(false)
const currentIndicator = ref(null)
const currentFile = ref(null)
const openPanels = ref([])

const { data: apiResponse, pending, refresh } = await useFetch(
    `${config.public.apiBase}/api/evaluator/assessments/${assignmentId}`, 
    { headers: { Authorization: `Bearer ${authStore.token}` } }
)

const assignmentInfo = computed(() => apiResponse.value?.assignment_info || {})
const evaluationForm = computed(() => apiResponse.value?.form_data || [])

const progress = computed(() => {
    if (!evaluationForm.value.length) return 0
    let total = 0, filled = 0
    evaluationForm.value.forEach(t => t.indicators.forEach(i => {
        total++
        if ((i.type === 'score_1_4' && i.score) || (i.type === 'yes_no'&& i.yes_no_val !== null)) filled++
    }))
    return total === 0 ? 0 : Math.round((filled / total) * 100)
})

const openEvidence = (indicator) => {
  currentIndicator.value = indicator
  currentFile.value = indicator.files?.[0] || null
  evidenceDialog.value = true
}

const submitEvaluation = async () => {
    try {
        const results = []
        evaluationForm.value.forEach(topic => {
            topic.indicators.forEach(item => {
                if (item.score || item.yes_no_val !== null || item.note) {
                    results.push({
                        indicator_id: item.id,
                        score: item.score,
                        value_yes_no: item.yes_no_val,
                        notes: item.note
                    })
                }
            })
        })

        const payload = {
            assignment_id: assignmentId,
            results: results,
            status: progress.value === 100 ? 'completed' : 'in_progress'
        }

        await $fetch(`${config.public.apiBase}/api/evaluator/submit`, {
            method: 'POST',
            headers: { Authorization: `Bearer ${authStore.token}` },
            body: payload
        })

        alert('บันทึกข้อมูลเรียบร้อย!')
        router.push('/evaluation_index') 
    } catch (err) {
        console.error(err)
        alert('เกิดข้อผิดพลาด: ' + (err.data?.message || err.message))
    }
}
</script>

<template>
    <div class="assess-bg" style="min-height: 100vh;">
        <!-- Loading State -->
        <div v-if="pending" class="loading-container">
            <v-card class="pa-8 rounded-xl text-center" elevation="12">
                <v-progress-circular indeterminate color="primary" size="64" width="6"></v-progress-circular>
                <div class="text-h6 text-grey-darken-1 mt-4">กำลังโหลดแบบประเมิน...</div>
            </v-card>
        </div>

        <!-- Main Content -->
        <div v-else>
            <!-- Header Section -->
            <v-sheet class="header-gradient px-6 px-md-12 py-8 mb-n6" elevation="0">
                <v-row align="center" justify="space-between" no-gutters>
                    <v-col cols="12" md="auto">
                        <div class="d-flex align-center mb-2">
                            <v-avatar color="white" size="80" class="mr-4 elevation-4">
                                <span class="text-h3 font-weight-black text-primary">
                                    {{ assignmentInfo.evaluatee?.charAt(0) || 'U' }}
                                </span>
                            </v-avatar>
                            <div>
                                <h1 class="text-h4 text-md-h3 font-weight-black text-white mb-1">
                                    {{ assignmentInfo.evaluatee }}
                                </h1>
                                <div class="text-subtitle-1 text-white d-flex align-center flex-wrap" style="opacity: 0.95">
                                    <v-icon size="20" class="mr-1">mdi-account-tie</v-icon>
                                    {{ assignmentInfo.position }}
                                    <v-divider vertical class="mx-3" style="opacity: 0.5"></v-divider>
                                    <v-icon size="20" class="mr-1">mdi-domain</v-icon>
                                    {{ assignmentInfo.department }}
                                </div>
                            </div>
                        </div>
                    </v-col>
                    <v-col cols="12" md="auto" class="text-center text-md-right mt-4 mt-md-0">
                        <v-card class="progress-card rounded-xl pa-4" elevation="8">
                            <div class="text-caption text-grey-darken-1 mb-2 font-weight-bold">ความคืบหน้า</div>
                            <div class="text-h3 font-weight-black mb-3" :class="progress === 100 ? 'text-success' : 'text-primary'">
                                {{ progress }}%
                            </div>
                            <v-progress-linear 
                                :model-value="progress" 
                                :color="progress === 100 ? 'success' : 'primary'" 
                                height="8" 
                                rounded
                            ></v-progress-linear>
                        </v-card>
                    </v-col>
                </v-row>
            </v-sheet>

            <!-- Content Container -->
            <v-container fluid class="px-6 px-md-12 pt-12 pb-16">
                <v-expansion-panels v-model="openPanels" multiple variant="accordion" class="evaluation-panels">
                    <v-expansion-panel 
                        v-for="topic in evaluationForm" 
                        :key="topic.id" 
                        elevation="6"
                        class="mb-4 rounded-xl topic-panel"
                    >
                        <!-- Panel Title -->
                        <v-expansion-panel-title class="panel-header py-4">
                            <div class="d-flex align-center w-100">
                                <v-avatar color="primary" size="48" class="mr-3">
                                    <v-icon color="white" size="28">mdi-clipboard-list</v-icon>
                                </v-avatar>
                                <div class="flex-grow-1">
                                    <div class="text-h6 font-weight-bold text-grey-darken-3">
                                        {{ topic.title }}
                                    </div>
                                    <div class="text-caption text-grey-darken-1 mt-1">
                                        <v-chip size="x-small" color="primary" variant="flat">{{ topic.code }}</v-chip>
                                        <span class="ml-2">{{ topic.indicators?.length || 0 }} รายการ</span>
                                    </div>
                                </div>
                            </div>
                        </v-expansion-panel-title>

                        <!-- Panel Content -->
                        <v-expansion-panel-text class="panel-content pa-0">
                            <div v-for="(item, idx) in topic.indicators" :key="item.id" class="indicator-item">
                                <v-card variant="flat" class="pa-6" :class="{ 'border-b': idx !== topic.indicators.length - 1 }">
                                    <v-row>
                                        <!-- Left: Indicator Info -->
                                        <v-col cols="12" lg="5">
                                            <div class="d-flex mb-3">
                                                <v-chip color="primary" size="small" variant="flat" class="mr-3 font-weight-bold">
                                                    {{ item.code }}
                                                </v-chip>
                                                <div class="flex-grow-1">
                                                    <div class="text-subtitle-1 font-weight-bold text-grey-darken-3 mb-2">
                                                        {{ item.name }}
                                                    </div>
                                                    <div class="text-body-2 text-grey-darken-1">
                                                        {{ item.desc }}
                                                    </div>
                                                </div>
                                            </div>

                                            <!-- Evidence Button -->
                                            <v-btn 
                                                v-if="item.files && item.files.length > 0" 
                                                variant="flat" 
                                                color="primary" 
                                                size="default"
                                                rounded="lg"
                                                class="evidence-btn"
                                                @click="openEvidence(item)"
                                            >
                                                <v-icon start>mdi-paperclip</v-icon>
                                                ดูหลักฐานแนบ ({{ item.files.length }})
                                            </v-btn>
                                            <v-chip 
                                                v-else 
                                                size="default" 
                                                variant="tonal" 
                                                color="grey"
                                            >
                                                <v-icon start>mdi-file-remove-outline</v-icon>
                                                ไม่มีไฟล์แนบ
                                            </v-chip>
                                        </v-col>

                                        <!-- Right: Scoring Section -->
                                        <v-col cols="12" lg="7">
                                            <v-card variant="outlined" class="score-card rounded-xl pa-5">
                                                <!-- Score 1-4 Type -->
                                                <div v-if="item.type === 'score_1_4'">
                                                    <div class="d-flex justify-space-between align-center mb-4">
                                                        <span class="text-body-1 font-weight-bold text-grey-darken-2">
                                                            <v-icon size="20" class="mr-1">mdi-star</v-icon>
                                                            ระดับคุณภาพ
                                                        </span>
                                                        <v-chip 
                                                            v-if="item.score" 
                                                            color="primary" 
                                                            variant="flat"
                                                            size="large"
                                                            class="font-weight-bold"
                                                        >
                                                            {{ item.score }} / 4 คะแนน
                                                        </v-chip>
                                                    </div>
                                                    <v-slider 
                                                        v-model="item.score" 
                                                        :min="1" 
                                                        :max="4" 
                                                        :step="1" 
                                                        show-ticks="always" 
                                                        tick-size="6" 
                                                        color="primary" 
                                                        thumb-label="always"
                                                        hide-details
                                                        class="custom-slider"
                                                    >
                                                        <template v-slot:thumb-label="{ modelValue }">
                                                            <div class="text-caption font-weight-bold">{{ modelValue }}</div>
                                                        </template>
                                                    </v-slider>
                                                    <div class="d-flex justify-space-between mt-2 text-caption text-grey">
                                                        <span>ต่ำสุด</span>
                                                        <span>สูงสุด</span>
                                                    </div>
                                                </div>

                                                <!-- Yes/No Type -->
                                                <div v-else-if="item.type === 'yes_no'">
                                                    <div class="text-body-1 font-weight-bold text-grey-darken-2 mb-3">
                                                        <v-icon size="20" class="mr-1">mdi-checkbox-marked-circle</v-icon>
                                                        ผลการประเมิน
                                                    </div>
                                                    <v-btn-toggle 
                                                        v-model="item.yes_no_val" 
                                                        density="comfortable" 
                                                        rounded="lg" 
                                                        mandatory 
                                                        class="w-100"
                                                        elevation="2"
                                                    >
                                                        <v-btn 
                                                            :value="1" 
                                                            color="success" 
                                                            class="flex-grow-1 py-4" 
                                                            variant="flat"
                                                            size="large"
                                                        >
                                                            <v-icon start>mdi-check-circle</v-icon>
                                                            ผ่าน
                                                        </v-btn>
                                                        <v-btn 
                                                            :value="0" 
                                                            color="error" 
                                                            class="flex-grow-1 py-4" 
                                                            variant="flat"
                                                            size="large"
                                                        >
                                                            <v-icon start>mdi-close-circle</v-icon>
                                                            ไม่ผ่าน
                                                        </v-btn>
                                                    </v-btn-toggle>
                                                </div>

                                                <!-- Note Field -->
                                                <v-divider class="my-4"></v-divider>
                                                <v-textarea 
                                                    v-model="item.note" 
                                                    density="comfortable" 
                                                    variant="outlined" 
                                                    label="ความคิดเห็นเพิ่มเติม"
                                                    placeholder="กรอกความคิดเห็นหรือข้อเสนอแนะ..."
                                                    rows="2"
                                                    hide-details
                                                    rounded="lg"
                                                    color="primary"
                                                >
                                                    <template v-slot:prepend-inner>
                                                        <v-icon>mdi-comment-text-outline</v-icon>
                                                    </template>
                                                </v-textarea>
                                            </v-card>
                                        </v-col>
                                    </v-row>
                                </v-card>
                            </div>
                        </v-expansion-panel-text>
                    </v-expansion-panel>
                </v-expansion-panels>
            </v-container>

            <!-- Fixed Footer -->
            <v-footer app class="footer-save" elevation="12">
                <v-container fluid class="px-6 px-md-12">
                    <v-row align="center" justify="space-between">
                        <v-col cols="12" md="auto" class="text-center text-md-left mb-3 mb-md-0">
                            <div class="text-body-2 text-grey-darken-1">
                                ความคืบหน้า: <strong class="text-primary">{{ progress }}%</strong>
                            </div>
                            <v-progress-linear 
                                :model-value="progress" 
                                :color="progress === 100 ? 'success' : 'primary'" 
                                height="6" 
                                rounded
                                class="mt-2"
                                style="max-width: 300px;"
                            ></v-progress-linear>
                        </v-col>
                        <v-col cols="12" md="auto" class="text-center">
                            <v-btn 
                                color="primary" 
                                size="x-large" 
                                rounded="lg"
                                elevation="4"
                                class="px-12 font-weight-bold btn-save"
                                @click="submitEvaluation"
                            >
                                <v-icon start size="24">mdi-content-save-check</v-icon>
                                บันทึกข้อมูล
                            </v-btn>
                        </v-col>
                    </v-row>
                </v-container>
            </v-footer>
        </div>

        <!-- Evidence Dialog -->
        <v-dialog v-model="evidenceDialog" max-width="1000" scrollable>
            <v-card v-if="currentIndicator" class="rounded-xl" elevation="24">
                <v-toolbar color="primary" dark class="px-4">
                    <v-avatar color="white" size="40" class="mr-3">
                        <v-icon color="primary">mdi-file-document</v-icon>
                    </v-avatar>
                    <v-toolbar-title class="text-h6 font-weight-bold">
                        {{ currentIndicator.code }}: หลักฐานประกอบ
                    </v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn 
                        v-if="currentFile" 
                        :href="currentFile.url" 
                        target="_blank" 
                        variant="flat" 
                        color="white"
                        class="mr-2"
                    >
                        <v-icon start>mdi-download</v-icon>
                        ดาวน์โหลด
                    </v-btn>
                    <v-btn icon="mdi-close" variant="text" @click="evidenceDialog = false"></v-btn>
                </v-toolbar>

                <v-card-text class="pa-0" style="min-height: 500px;">
                    <v-row no-gutters>
                        <!-- File List -->
                        <v-col cols="12" md="4" class="file-list-col pa-4">
                            <div class="text-subtitle-2 font-weight-bold mb-3 text-grey-darken-2">
                                <v-icon size="18" class="mr-1">mdi-file-multiple</v-icon>
                                รายการไฟล์ ({{ currentIndicator.files?.length || 0 }})
                            </div>
                            <v-list density="comfortable" rounded="lg">
                                <v-list-item 
                                    v-for="(file, i) in currentIndicator.files" 
                                    :key="i" 
                                    :title="file.name"
                                    :prepend-icon="file.type === 'pdf' ? 'mdi-file-pdf-box' : 'mdi-image'" 
                                    :value="file" 
                                    color="primary" 
                                    rounded="lg" 
                                    class="mb-2 file-item"
                                    :class="{ 'active-file': currentFile === file }"
                                    @click="currentFile = file"
                                ></v-list-item>
                            </v-list>
                        </v-col>

                        <!-- File Preview -->
                        <v-col cols="12" md="8" class="preview-col">
                            <div v-if="!currentFile" class="preview-placeholder">
                                <v-icon size="80" color="grey-lighten-1">mdi-eye-off-outline</v-icon>
                                <div class="text-h6 text-grey-darken-1 mt-4">เลือกไฟล์เพื่อดูตัวอย่าง</div>
                            </div>
                            <v-img 
                                v-else-if="currentFile.type === 'image'" 
                                :src="currentFile.url" 
                                height="100%" 
                                contain
                            ></v-img>
                            <iframe 
                                v-else-if="currentFile.type === 'pdf'" 
                                :src="currentFile.url" 
                                width="100%" 
                                height="100%" 
                                style="border: none;"
                            ></iframe>
                            <div v-else class="preview-placeholder">
                                <v-icon size="80" color="grey-lighten-1">mdi-file-question</v-icon>
                                <div class="text-h6 text-grey-darken-1 mt-4">ไม่รองรับการแสดงผล</div>
                            </div>
                        </v-col>
                    </v-row>
                </v-card-text>
            </v-card>
        </v-dialog>
    </div>
</template>

<style scoped>
/* Background */
.assess-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Loading Container */
.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Header Gradient */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

/* Progress Card */
.progress-card {
  background: white;
  min-width: 200px;
}

/* Evaluation Panels */
.evaluation-panels :deep(.v-expansion-panel) {
  border: 2px solid rgba(102, 126, 234, 0.1);
  transition: all 0.3s ease;
}

.evaluation-panels :deep(.v-expansion-panel:hover) {
  border-color: rgba(102, 126, 234, 0.3);
  transform: translateY(-2px);
}

.panel-header {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.05) 0%, rgba(118, 75, 162, 0.05) 100%);
}

.panel-content {
  background: white;
}

/* Indicator Items */
.indicator-item {
  transition: background 0.2s ease;
}

.indicator-item:hover {
  background: rgba(102, 126, 234, 0.02);
}

.border-b {
  border-bottom: 2px solid rgba(0, 0, 0, 0.06);
}

/* Evidence Button */
.evidence-btn {
  transition: all 0.3s ease;
}

.evidence-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

/* Score Card */
.score-card {
  border: 2px solid rgba(102, 126, 234, 0.15);
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.02) 0%, rgba(118, 75, 162, 0.02) 100%);
  transition: all 0.3s ease;
}

.score-card:hover {
  border-color: rgba(102, 126, 234, 0.3);
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.1);
}

/* Custom Slider */
.custom-slider :deep(.v-slider-thumb) {
  border: 3px solid white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

/* Footer Save */
.footer-save {
  background: white;
  border-top: 2px solid rgba(102, 126, 234, 0.1);
}

.btn-save {
  transition: all 0.3s ease;
}

.btn-save:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(102, 126, 234, 0.4) !important;
}

/* Evidence Dialog */
.file-list-col {
  background: #f5f7fa;
  border-right: 1px solid rgba(0, 0, 0, 0.08);
}

.file-item {
  transition: all 0.2s ease;
}

.file-item:hover {
  transform: translateX(4px);
}

.active-file {
  background: rgba(102, 126, 234, 0.1);
}

.preview-col {
  background: #2c2c2c;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 500px;
}

.preview-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  height: 100%;
}
</style>