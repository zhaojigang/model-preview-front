<template>
  <div class="model-preview-container">
    <!-- 筛选条件 -->
    <div class="filter-section">
      <div class="filter-content">
        <div class="filter-options">
          <div 
            v-for="option in modelOptions" 
            :key="option.value"
            :class="['filter-item', { active: modelType === option.value }]"
            @click="modelType = option.value"
          >
            {{ option.label }}
          </div>
        </div>
        
        <!-- 搜索框 -->
        <el-input
          v-model="searchKeyword"
          placeholder="搜索模型名称"
          class="search-input"
          clearable
        >
          <template #prefix>
            <el-icon class="search-icon"><Search /></el-icon>
          </template>
        </el-input>
      </div>
    </div>

    <!-- 瀑布流展示区 -->
    <div class="waterfall-container">
      <el-row :gutter="16">
        <el-col 
          v-for="item in filteredModelList" 
          :key="item.name" 
          :xs="24"
          :sm="12"
          :md="8"
          :lg="6"
          :xl="4"
          class="model-col"
        >
          <div class="model-card" @click="showModelDetail(item)">
            <el-image 
              :src="item.pic" 
              fit="cover"
              class="model-image"
            />
            <div class="model-name">{{ item.name }}</div>
          </div>
        </el-col>
      </el-row>
    </div>

    <!-- 详情弹窗 -->
    <el-dialog
      v-model="dialogVisible"
      :title="currentModel?.name"
      width="480px"
      :show-close="false"
      class="model-detail-dialog"
    >
      <div class="dialog-content">
        <div class="usage-tips">
          <div class="tips-title">使用说明</div>
          <pre>{{ currentModel?.useTips }}</pre>
        </div>
        <div class="download-section">
          <el-button 
            class="download-btn" 
            @click="handleDownload"
          >
            <i class="el-icon-download" />
            获取模型
          </el-button>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import axios from 'axios'
import { ElMessage } from 'element-plus'
import { Search } from '@element-plus/icons-vue'

const modelType = ref('checkpoint')
const modelList = ref([])
const dialogVisible = ref(false)
const currentModel = ref(null)
const searchKeyword = ref('')

const modelOptions = [
  { label: 'Checkpoint', value: 'checkpoint' },
  { label: 'DiffusionModel', value: 'diffusionModel' },
  { label: 'Lora', value: 'lora' }
]

// 过滤后的模型列表
const filteredModelList = computed(() => {
  if (!searchKeyword.value) return modelList.value
  
  const keyword = searchKeyword.value.toLowerCase()
  return modelList.value.filter(model => 
    model.name.toLowerCase().includes(keyword)
  )
})

// 将文件路径转换为 base64
const convertToBase64 = async (filePath) => {
  try {
    const response = await fetch(`file://${filePath}`)
    const blob = await response.blob()
    return new Promise((resolve, reject) => {
      const reader = new FileReader()
      reader.onloadend = () => resolve(reader.result)
      reader.onerror = reject
      reader.readAsDataURL(blob)
    })
  } catch (error) {
    console.error('图片转换失败：', error)
    return ''
  }
}

// 获取模型列表
const fetchModelList = async () => {
  try {
    const response = await axios.get(`/api/modelPreviewInfos`, {
      params: {
        modelFirstType: modelType.value
      }
    })
    // 修改图片路径
    modelList.value = response.data.map(item => ({
      ...item,
      pic: `/api/images?path=${encodeURIComponent(item.pic)}`
    }))
  } catch (error) {
    console.error('获取模型列表失败：', error)
    ElMessage.error('获取模型列表失败')
  }
}

// 监听模型类型变化
watch(modelType, () => {
  fetchModelList()
})

// 显示模型详情
const showModelDetail = (model) => {
  currentModel.value = model
  dialogVisible.value = true
}

// 处理下载
const handleDownload = () => {
  if (currentModel.value?.link) {
    window.open(currentModel.value.link, '_blank')
    dialogVisible.value = false
    currentModel.value = null
  }
}

// 关闭弹窗
const handleClose = () => {
  dialogVisible.value = false
  currentModel.value = null
}

onMounted(() => {
  fetchModelList()
})
</script>

<style scoped>
.model-preview-container {
  padding: 16px;
  background-color: #f5f7fa;
  min-height: 100vh;
}

.filter-section {
  background: #fff;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.05);
  margin-bottom: 16px;
}

.filter-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 16px;
}

.filter-options {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  flex: 1;
}

.filter-item {
  padding: 6px 16px;
  border-radius: 16px;
  cursor: pointer;
  font-size: 14px;
  color: #606266;
  background: #f5f7fa;
  transition: all 0.3s ease;
  border: 1px solid transparent;
}

.filter-item:hover {
  background: #f0f0f0;
  border-color: #d4d4d4;
}

.filter-item.active {
  background: linear-gradient(135deg, #24242a 0%, #2c2c35 100%);
  color: #d4af37;
  border-color: #d4af37;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.waterfall-container {
  background: #fff;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.05);
  margin-top: 0;
}

.model-col {
  margin-bottom: 16px;
}

.model-card {
  background: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.3s;
  width: 256px;
  margin: 0 auto;
}

.model-card:hover {
  transform: translateY(-5px);
}

.model-image {
  width: 256px;
  height: 256px;
  object-fit: contain;
  background-color: #f5f7fa;
}

.model-name {
  padding: 10px;
  text-align: center;
  font-size: 14px;
  color: #333;
  word-break: break-word;
  overflow-wrap: break-word;
  line-height: 1.4;
  max-height: 2.8em;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

/* 弹窗样式优化 */
:deep(.model-detail-dialog .el-dialog) {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

:deep(.model-detail-dialog .el-dialog__header) {
  margin: 0;
  padding: 24px 24px 20px;
  background: #fff;
  border-bottom: 1px solid #eee;
  text-align: center;
  max-width: 100%;
}

:deep(.model-detail-dialog .el-dialog__title) {
  color: #24242a;
  font-size: 18px;
  font-weight: 500;
  background: linear-gradient(135deg, #24242a 0%, #2c2c35 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  display: inline-block;
  word-break: break-word;
  white-space: pre-wrap;
  line-height: 1.4;
  max-width: 100%;
  padding: 0 20px;
}

:deep(.model-detail-dialog .el-dialog__headerbtn),
:deep(.model-detail-dialog .el-dialog__close),
:deep(.model-detail-dialog .el-dialog__close:hover) {
  display: none;
}

:deep(.model-detail-dialog .el-dialog__body) {
  padding: 24px;
  background: #fff;
}

.dialog-content {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.usage-tips {
  background: #fafafa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #eee;
  font-family: "PingFang SC", "Microsoft YaHei", sans-serif;
}

.tips-title {
  font-size: 16px;
  font-weight: 500;
  color: #24242a;
  margin-bottom: 12px;
  padding-bottom: 12px;
  border-bottom: 1px solid #eee;
}

.usage-tips pre {
  margin: 0;
  white-space: pre-wrap;
  word-wrap: break-word;
  line-height: 1.8;
  font-size: 14px;
  color: #606266;
  font-family: inherit;
}

.download-section {
  text-align: center;
  padding-top: 8px;
}

.download-btn {
  background: linear-gradient(135deg, #24242a 0%, #2c2c35 100%);
  border-color: #d4af37;
  color: #d4af37;
  padding: 12px 36px;
  font-size: 15px;
  border-radius: 24px;
  transition: all 0.3s ease;
}

.download-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  background: linear-gradient(135deg, #2c2c35 0%, #24242a 100%);
}

.download-btn i {
  margin-right: 8px;
}

@media screen and (max-width: 576px) {
  .model-card {
    width: 100%;
  }
  
  .model-image {
    width: 100%;
    height: 256px;
    max-width: 256px;
    margin: 0 auto;
  }
}

.search-input {
  width: 240px;
  flex-shrink: 0;
}

.search-input :deep(.el-input__wrapper) {
  border-radius: 24px;
  box-shadow: 0 0 0 1px #e4e7ed;
  padding-left: 4px;
}

.search-input :deep(.el-input__wrapper.is-focus) {
  box-shadow: 0 0 0 1px #d4af37;
}

.search-input :deep(.el-input__inner) {
  height: 40px;
  font-size: 14px;
  color: #24242a;
}

.search-input :deep(.el-input__inner::placeholder) {
  color: #909399;
}

.search-icon {
  font-size: 16px;
  color: #909399;
  margin: 0 4px;
}

@media screen and (max-width: 768px) {
  .filter-content {
    flex-direction: column;
    align-items: stretch;
  }

  .search-input {
    width: 100%;
  }
}
</style> 