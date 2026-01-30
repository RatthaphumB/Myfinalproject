<template>
  <div class="evidence-bg" style="min-height: 100vh;">
    <v-overlay :model-value="loading" class="align-center justify-center" style="z-index: 9999;">
      <v-card class="pa-8 rounded-xl text-center" elevation="12">
        <v-progress-circular color="primary" indeterminate size="64" width="6"></v-progress-circular>
        <div class="text-h6 text-grey-darken-1 mt-4">กำลังโหลดข้อมูล...</div>
      </v-card>
    </v-overlay>

    <v-sheet class="header-gradient px-4 px-md-12 py-8 mb-n8 pb-12" elevation="0">
      <v-row align="center" justify="space-between" no-gutters>
        <v-col cols="12" md="auto">
          <div class="d-flex flex-column flex-md-row align-center text-center text-md-left mb-2">
            <v-avatar color="white" size="80" class="mb-4 mb-md-0 mr-md-6 elevation-4">
              <span class="text-h4 font-weight-black text-primary">{{ userInitials }}</span>
            </v-avatar>
            <div>
              <h1 class="text-h5 text-md-h3 font-weight-black text-white mb-2">
                ส่งผลงานการประเมิน
              </h1>
              <div class="text-subtitle-1 text-white d-flex align-center justify-center justify-md-start flex-wrap" style="opacity: 0.95">
                <v-icon size="20" class="mr-1">mdi-account</v-icon>
                <span class="mr-2">{{ currentUser.name_th }}</span>
                <span class="d-none d-md-block mx-2">|</span>
                <div class="w-100 d-md-none my-1"></div> <v-icon size="20" class="mr-1">mdi-domain</v-icon>
                แผนก: {{ currentUser.dept_id || '-' }}
              </div>
            </div>
          </div>
        </v-col>
        <v-col cols="12" md="auto" class="text-center text-md-right mt-4 mt-md-0">
          <v-chip color="white" size="large" variant="flat" class="font-weight-bold" v-if="currentPeriodName">
            <v-icon start color="primary">mdi-calendar-clock</v-icon>
            {{ currentPeriodName }}
          </v-chip>
        </v-col>
      </v-row>
    </v-sheet>

    <v-container fluid class="px-4 px-md-12 pt-8 pb-8 content-container" v-if="!loading">
      
      <v-card v-if="evaluationData.length === 0" elevation="8" rounded="xl" class="pa-8 pa-md-12 text-center mt-6">
        <v-avatar color="grey-lighten-3" size="100" class="mb-6">
          <v-icon color="grey-lighten-1" size="60">mdi-file-document-alert</v-icon>
        </v-avatar>
        <div class="text-h6 text-md-h5 text-grey-darken-2 font-weight-bold mb-2">
          ไม่พบรายการประเมิน
        </div>
        <div class="text-body-2 text-md-body-1 text-grey">
          ขณะนี้ไม่มีรายการประเมินสำหรับปีการศึกษานี้
        </div>
      </v-card>

      <v-expansion-panels v-else multiple variant="accordion" class="evaluation-panels mt-4">
        <v-expansion-panel 
          v-for="(topic, tIndex) in evaluationData" 
          :key="topic.id" 
          elevation="4"
          class="mb-4 rounded-xl topic-panel"
        >
          <v-expansion-panel-title class="panel-header py-4 px-4">
            <div class="d-flex flex-column flex-sm-row align-start align-sm-center w-100">
              <div class="d-flex align-center flex-grow-1 mb-2 mb-sm-0 mr-sm-4" style="overflow: hidden;">
                <v-avatar color="primary" size="36" class="mr-3 flex-shrink-0">
                  <v-icon color="white" size="20">mdi-clipboard-text</v-icon>
                </v-avatar>
                <div style="min-width: 0;"> <div class="text-subtitle-1 text-md-h6 font-weight-bold text-grey-darken-3 text-truncate-2">
                    {{ topic.title_th }}
                  </div>
                  <div class="text-caption text-grey-darken-1 mt-0">
                    <v-icon size="12">mdi-format-list-bulleted</v-icon>
                    {{ topic.indicators?.length || 0 }} รายการ
                  </div>
                </div>
              </div>

              <v-chip 
                size="small" 
                :color="isTopicComplete(topic) ? 'success' : 'warning'" 
                variant="flat"
                class="font-weight-bold align-self-start align-self-sm-center ml-12 ml-sm-0"
              >
                <v-icon start size="16">
                  {{ isTopicComplete(topic) ? 'mdi-check-circle' : 'mdi-clock-outline' }}
                </v-icon>
                {{ getTopicProgress(topic) }}
              </v-chip>
            </div>
          </v-expansion-panel-title>

          <v-expansion-panel-text class="panel-content px-0">
            <v-list lines="two" class="pa-0">
              <template v-for="(indicator, iIndex) in topic.indicators" :key="indicator.id">
                <v-card 
                  variant="outlined" 
                  class="mb-4 indicator-card rounded-lg mx-1 mx-md-2"
                  :class="{ 'completed-card': indicator.uploaded_files && indicator.uploaded_files.length > 0 }"
                >
                  <v-card-text class="pa-3 pa-md-4">
                    <div class="mb-3">
                      <div class="d-flex align-start mb-2">
                        <v-chip size="x-small" color="primary" variant="flat" class="mr-2 mt-1 font-weight-bold flex-shrink-0">
                          {{ indicator.code }}
                        </v-chip>
                        <div class="text-subtitle-2 text-md-subtitle-1 font-weight-bold text-grey-darken-3" style="line-height: 1.4;">
                          {{ indicator.name_th }}
                        </div>
                      </div>
                      <div class="text-caption text-md-body-2 text-grey-darken-1 pl-1">
                        {{ indicator.description }}
                      </div>
                    </div>

                    <div 
                      v-if="indicator.uploaded_files && indicator.uploaded_files.length > 0"
                      class="uploaded-files-section mb-4 pa-3 rounded-lg"
                    >
                      <div class="text-caption font-weight-bold text-success mb-2 d-flex align-center">
                        <v-icon size="16" class="mr-1">mdi-check-circle</v-icon>
                        ไฟล์ที่อัปโหลดแล้ว
                      </div>
                      <div class="d-flex flex-wrap gap-2">
                        <v-chip 
                          v-for="file in indicator.uploaded_files" 
                          :key="file.id" 
                          color="success" 
                          variant="flat"
                          size="small"
                          closable 
                          @click:close="deleteFile(file.id, indicator)"
                          class="file-chip"
                          style="max-width: 100%;"
                        >
                          <template v-slot:prepend>
                            <v-icon size="16" start>mdi-file-check</v-icon>
                          </template>
                          <span class="text-truncate" style="max-width: 180px;">{{ file.file_name }}</span>
                        </v-chip>
                      </div>
                    </div>

                    <v-card variant="tonal" color="grey-lighten-4" class="pa-3 pa-md-4 rounded-lg upload-section">
                      <div class="d-flex flex-column gap-3">
                        
                        <div v-if="indicator.allowed_evidence.length > 1">
                          <div class="text-caption font-weight-bold text-grey-darken-2 mb-1">
                            <v-icon size="14" class="mr-1">mdi-file-document</v-icon>
                            เลือกประเภทเอกสาร
                          </div>
                          <v-select 
                            v-model="indicator.selectedEvidenceType"
                            :items="indicator.allowed_evidence" 
                            item-title="name_th" 
                            item-value="id" 
                            placeholder="เลือกประเภท"
                            density="compact" 
                            variant="outlined" 
                            hide-details
                            rounded="lg"
                            color="primary"
                            bg-color="white"
                          ></v-select>
                        </div>

                        <div v-else class="d-flex align-center flex-wrap">
                          <span class="text-caption text-md-body-2 font-weight-medium text-grey-darken-2 mr-2">
                            ประเภท:
                          </span>
                          <v-chip size="x-small" variant="flat" color="primary" class="my-1">
                            {{ indicator.allowed_evidence[0]?.name_th || 'เอกสาร' }}
                          </v-chip>
                        </div>

                        <div>
                          <div class="text-caption font-weight-bold text-grey-darken-2 mb-1">
                            <v-icon size="14" class="mr-1">mdi-upload</v-icon>
                            อัปโหลดไฟล์
                          </div>
                          <v-file-input 
                            v-model="indicator.tempFile" 
                            :loading="uploadingId === indicator.id"
                            :disabled="uploadingId === indicator.id" 
                            label="เลือกไฟล์..." 
                            prepend-icon=""
                            prepend-inner-icon="mdi-paperclip" 
                            variant="outlined" 
                            density="compact" 
                            hide-details 
                            rounded="lg"
                            color="primary"
                            bg-color="white"
                            @update:modelValue="handleFileUpload(indicator)"
                            class="custom-file-input"
                          >
                            <template v-slot:selection="{ fileNames }">
                              <span class="text-caption text-truncate">{{ fileNames[0] }}</span>
                            </template>
                            <template v-slot:append-inner>
                              <v-btn 
                                v-if="indicator.tempFile"
                                icon="mdi-upload"
                                size="x-small"
                                color="primary"
                                variant="flat"
                                class="ml-1"
                              ></v-btn>
                            </template>
                          </v-file-input>
                        </div>
                      </div>
                    </v-card>
                  </v-card-text>
                </v-card>
              </template>
            </v-list>
          </v-expansion-panel-text>
        </v-expansion-panel>
      </v-expansion-panels>

    </v-container>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "EvidenceSubmission",
  data() {
    return {
      currentUser: { name_th: 'User', role: '', dept_id: '', id: null },
      currentPeriodId: null, 
      currentPeriodName: '', 
      token: '',
      loading: true,
      uploadingId: null,
      evaluationData: []
    };
  },
  computed: {
    userInitials() {
      return this.currentUser.name_th ? this.currentUser.name_th.charAt(0) : 'U';
    }
  },
  mounted() {
    this.initPage();
  },
  methods: {
    getTopicProgress(topic) {
      if (!topic.indicators) return '0/0';
      const completed = topic.indicators.filter(i => i.uploaded_files && i.uploaded_files.length > 0).length;
      return `${completed}/${topic.indicators.length}`;
    },
    isTopicComplete(topic) {
      if (!topic.indicators) return false;
      const completed = topic.indicators.filter(i => i.uploaded_files && i.uploaded_files.length > 0).length;
      return completed === topic.indicators.length && topic.indicators.length > 0;
    },
    parseJwt(token) {
      try {
        const base64Url = token.split('.')[1];
        const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
        const jsonPayload = decodeURIComponent(window.atob(base64).split('').map(function (c) {
          return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
        }).join(''));
        return JSON.parse(jsonPayload);
      } catch (e) {
        return {};
      }
    },

    async initPage() {
      const token = localStorage.getItem('token');
      if (!token) {
        alert('กรุณาเข้าสู่ระบบก่อน');
        this.$router.push('/login');
        return;
      }
      this.token = token;

      const userStr = localStorage.getItem('user_info');
      if (userStr) {
        try { this.currentUser = JSON.parse(userStr); } catch (e) { }
      }
      if (!this.currentUser.id) {
        const decoded = this.parseJwt(token);
        this.currentUser = {
          id: decoded.id || decoded.sub,
          name_th: decoded.name || decoded.username || 'ผู้ใช้งาน',
          dept_id: decoded.dept_id || '-',
          role: decoded.role || 'user'
        };
      }

      try {
        // Mock data logic or API call
        const periodRes = await axios.get('http://localhost:7000/api/evaluatee/current-period');
        this.currentPeriodId = periodRes.data.id;
        this.currentPeriodName = periodRes.data.period_name;

        await this.fetchEvaluationData();

      } catch (error) {
        console.error("Failed to get period:", error);
        // Fallback for demo if API fails
        this.loading = false;
      }
    },

    async fetchEvaluationData() {
      try {
        const response = await axios.get('http://localhost:7000/api/evaluatee/form-data', {
          headers: { Authorization: `Bearer ${this.token}` },
          params: { period_id: this.currentPeriodId } 
        });
        this.evaluationData = response.data;
        this.processEvaluationData();
      } catch (error) {
        console.error("Fetch Error:", error);
        // Handle auth error
      } finally {
        this.loading = false;
      }
    },

    processEvaluationData() {
      this.evaluationData.forEach(topic => {
        topic.indicators.forEach(ind => {
          ind.tempFile = null;
          if (ind.allowed_evidence && ind.allowed_evidence.length === 1) {
            ind.selectedEvidenceType = ind.allowed_evidence[0].id;
          } else {
            ind.selectedEvidenceType = null;
          }
        });
      });
    },

    async handleFileUpload(indicator) {
      const file = Array.isArray(indicator.tempFile) ? indicator.tempFile[0] : indicator.tempFile;
      if (!file) return;

      if (!indicator.selectedEvidenceType) {
        alert('กรุณาเลือกประเภทเอกสารก่อน');
        indicator.tempFile = null;
        return;
      }

      this.uploadingId = indicator.id;

      const formData = new FormData();
      formData.append('file', file);
      formData.append('period_id', this.currentPeriodId); 
      formData.append('evaluatee_id', this.currentUser.id);
      formData.append('indicator_id', indicator.id);
      formData.append('evidence_type_id', indicator.selectedEvidenceType);

      try {
        const res = await axios.post('http://localhost:7000/api/upload/file', formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
            'Authorization': `Bearer ${this.token}`
          }
        });
        
        if (!indicator.uploaded_files) indicator.uploaded_files = [];
        indicator.uploaded_files.push({
         id: res.data.attachment_id || res.data.id || Date.now(),
         file_name: file.name
        });
        indicator.tempFile = null;

      } catch (error) {
        alert('อัปโหลดล้มเหลว: ' + (error.response?.data?.message || error.message));
        indicator.tempFile = null;
      } finally {
        this.uploadingId = null;
      }
    },

    async deleteFile(fileId, indicator) {
      if (!confirm('ต้องการลบไฟล์นี้ใช่ไหม?')) return;

      try {
        await axios.delete(`http://localhost:7000/api/upload/file/${fileId}`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        indicator.uploaded_files = indicator.uploaded_files.filter(f => f.id !== fileId);
      } catch (error) {
        console.error("Delete Error:", error);
        alert('ลบไฟล์ไม่สำเร็จ: ' + (error.response?.data?.message || error.message));
      }
    }
  }
};
</script>

<style scoped>
/* Background */
.evidence-bg {
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef3 100%);
}

/* Header Gradient */
.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
  position: relative;
  overflow: hidden;
}

/* Header pattern decoration (Optional) */
.header-gradient::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at top right, rgba(255,255,255,0.1) 0%, transparent 60%);
}

/* Evaluation Panels */
.evaluation-panels :deep(.v-expansion-panel) {
  border: 1px solid rgba(102, 126, 234, 0.1);
  transition: all 0.3s ease;
  overflow: hidden;
}

.evaluation-panels :deep(.v-expansion-panel:hover) {
  border-color: rgba(102, 126, 234, 0.3);
}

/* Panel Header */
.panel-header {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.05) 0%, rgba(118, 75, 162, 0.05) 100%);
}

/* Panel Content */
.panel-content {
  background: white;
}

/* Indicator Card */
.indicator-card {
  border: 1px solid rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
}

.indicator-card:hover {
  border-color: rgba(102, 126, 234, 0.3);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.1);
}

.completed-card {
  border-color: rgba(76, 175, 80, 0.3);
  background: linear-gradient(135deg, rgba(76, 175, 80, 0.01) 0%, rgba(76, 175, 80, 0.03) 100%);
}

/* Uploaded Files Section */
.uploaded-files-section {
  background: linear-gradient(135deg, rgba(76, 175, 80, 0.08) 0%, rgba(76, 175, 80, 0.12) 100%);
  border: 1px solid rgba(76, 175, 80, 0.2);
}

.file-chip {
  transition: all 0.2s ease;
  max-width: 100%;
}

.file-chip:hover {
  transform: translateY(-2px);
}

/* Upload Section */
.upload-section {
  border: 2px dashed rgba(102, 126, 234, 0.2);
  transition: all 0.3s ease;
}

.upload-section:hover {
  border-color: rgba(102, 126, 234, 0.4);
  background: rgba(102, 126, 234, 0.05) !important;
}

/* Text Truncate Helper */
.text-truncate-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  line-height: 1.3;
}

/* Mobile Adjustments using CSS */
@media (max-width: 600px) {
  :deep(.v-expansion-panel-title__overlay) {
    opacity: 0 !important; /* Cleaner look on touch */
  }
  
  .topic-panel {
    border-radius: 16px !important;
  }

  .content-container {
    padding-left: 12px !important;
    padding-right: 12px !important;
  }
}

/* Scrollbar */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background-color: rgba(102, 126, 234, 0.3);
  border-radius: 4px;
}
</style>