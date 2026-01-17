<template>
  <div class="csv-uploader">
    <!-- 上传区域 -->
    <div class="upload-area" @drop="handleDrop" @dragover.prevent @dragenter.prevent>
      <div class="upload-content">
        <div class="upload-icon">
          <Icon name="upload" size="2xl" />
        </div>
        <p class="upload-text">拖拽文件到此处</p>
        <p class="upload-subtext">或 <span class="upload-link" @click="triggerFileInput">点击上传</span></p>
        <p class="upload-tip">支持 CSV 格式，第一行为列名</p>
        <input
          ref="fileInput"
          type="file"
          accept=".csv"
          @change="handleFileChange"
          class="hidden-input"
        />
      </div>
    </div>

    <!-- 列映射配置 -->
    <template v-if="csvHeaders.length > 0">
      <div class="mapping-config">
        <div class="section-header">
          <h3>
            <Icon name="clipboard-list" size="lg" />
            <span>列映射配置</span>
          </h3>
          <button class="auto-btn" @click="autoMap">
            <Icon name="sparkles" size="sm" />
            <span>自动识别</span>
          </button>
        </div>
        <div class="mapping-fields">
          <div class="mapping-field">
            <label class="field-label required">姓名列</label>
            <select v-model="columnMapping.name" class="field-select" @change="applyMapping">
              <option value="">-- 请选择 --</option>
              <option
                v-for="(header, index) in csvHeaders"
                :key="index"
                :value="header"
              >
                {{ header }} (列{{ index + 1 }})
              </option>
            </select>
          </div>
          <div class="mapping-field">
            <label class="field-label">工号列</label>
            <select v-model="columnMapping.id" class="field-select" @change="applyMapping">
              <option value="">-- 不使用 --</option>
              <option
                v-for="(header, index) in csvHeaders"
                :key="index"
                :value="header"
              >
                {{ header }} (列{{ index + 1 }})
              </option>
            </select>
          </div>
          <div class="mapping-field">
            <label class="field-label">手机号列</label>
            <select v-model="columnMapping.phone" class="field-select" @change="applyMapping">
              <option value="">-- 不使用 --</option>
              <option
                v-for="(header, index) in csvHeaders"
                :key="index"
                :value="header"
              >
                {{ header }} (列{{ index + 1 }})
              </option>
            </select>
          </div>
          <div class="mapping-field">
            <label class="field-label">部门列</label>
            <select v-model="columnMapping.department" class="field-select" @change="applyMapping">
              <option value="">-- 不使用 --</option>
              <option
                v-for="(header, index) in csvHeaders"
                :key="index"
                :value="header"
              >
                {{ header }} (列{{ index + 1 }})
              </option>
            </select>
          </div>
          <div class="mapping-field">
            <label class="field-label">权重列</label>
            <select v-model="columnMapping.weight" class="field-select" @change="applyMapping">
              <option value="">-- 不使用 --</option>
              <option
                v-for="(header, index) in csvHeaders"
                :key="index"
                :value="header"
              >
                {{ header }} (列{{ index + 1 }})
              </option>
            </select>
          </div>
        </div>
      </div>
    </template>

    <!-- 预览名单 -->
    <template v-if="participants.length > 0">
      <div class="preview-section">
        <div class="section-header">
          <h3>
            <Icon name="users" size="lg" />
            <span>名单预览 (共 {{ participants.length }} 人)</span>
          </h3>
          <div class="actions">
            <button class="action-btn" @click="checkDuplicates">
              <Icon name="search" size="sm" />
              <span>检查重复</span>
            </button>
            <button class="action-btn" @click="loadSampleData">
              <Icon name="clipboard-list" size="sm" />
              <span>示例数据</span>
            </button>
            <button class="action-btn" @click="saveData">
              <Icon name="save" size="sm" />
              <span>保存</span>
            </button>
            <button class="action-btn danger" @click="clearData">
              <Icon name="trash" size="sm" />
              <span>清空</span>
            </button>
          </div>
        </div>

        <!-- 重复检查结果 -->
        <div v-if="duplicateResult" class="duplicate-alert" :class="duplicateResult.hasDuplicate ? 'warning' : 'success'">
          <div class="alert-header">
            <Icon :name="duplicateResult.hasDuplicate ? 'help-circle' : 'check'" size="md" class="alert-icon" />
            <span class="alert-title">{{ duplicateResult.hasDuplicate ? '发现重复数据' : '无重复数据' }}</span>
            <button v-if="duplicateResult.hasDuplicate" class="remove-duplicate-btn" @click="removeDuplicates">
              <Icon name="trash" size="sm" />
              <span>移除重复项</span>
            </button>
            <button class="alert-close" @click="duplicateResult = null">
              <Icon name="x" size="sm" />
            </button>
          </div>
          <div v-if="duplicateResult.hasDuplicate" class="alert-body">
            <div v-if="duplicateResult.duplicateIds.length > 0" class="duplicate-group">
              <span class="duplicate-label">重复工号：</span>
              <span class="duplicate-items">{{ duplicateResult.duplicateIds.join('、') }}</span>
            </div>
            <div v-if="duplicateResult.duplicateNames.length > 0" class="duplicate-group">
              <span class="duplicate-label">重复姓名：</span>
              <span class="duplicate-items">{{ duplicateResult.duplicateNames.join('、') }}</span>
            </div>
          </div>
        </div>

        <div class="participant-grid">
          <div
            v-for="(p, index) in displayParticipants"
            :key="index"
            class="participant-card"
          >
            <div class="card-header">
              <div class="participant-avatar">{{ p.name?.[0] || '?' }}</div>
              <button class="remove-btn" @click="removeParticipant(index)">
                <Icon name="x" size="sm" />
              </button>
            </div>
            <div class="card-body">
              <div class="participant-name">{{ p.name }}</div>
              <div v-if="p.id" class="participant-info">工号: {{ p.id }}</div>
              <div v-if="p.phone" class="participant-info">手机: {{ p.phone }}</div>
              <div v-if="p.department" class="participant-info">{{ p.department }}</div>
            </div>
          </div>
        </div>

        <div v-if="participants.length > 20" class="load-more">
          <button class="more-btn" @click="showAll = !showAll">
            {{ showAll ? '收起' : `查看剩余 ${participants.length - 20} 人` }}
          </button>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { useLotteryStore } from '@/stores/lottery'
import { parseCSV, transformToParticipants, autoDetectColumns } from '@/utils/csv'
import Icon from '@/components/common/Icon.vue'

const store = useLotteryStore()

// 组件挂载时，如果没有参与者数据，自动加载示例数据
onMounted(() => {
  if (store.participants.length === 0) {
    loadSampleDataSilent()
  }
})

const csvHeaders = computed(() => store.csvHeaders)
const participants = computed(() => store.participants)
const columnMapping = computed({
  get: () => store.columnMapping,
  set: (val) => store.setColumnMapping(val),
})

const showAll = ref(false)
const rawCsvData = ref<string[][]>([])
const fileInput = ref<HTMLInputElement | null>(null)

// 重复检查结果
const duplicateResult = ref<{
  hasDuplicate: boolean
  duplicateIds: string[]
  duplicateNames: string[]
} | null>(null)

// 显示的参与者列表（分页）
const displayParticipants = computed(() => {
  return showAll.value ? participants.value : participants.value.slice(0, 20)
})

function triggerFileInput() {
  fileInput.value?.click()
}

async function handleFileChange(event: Event) {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) {
    processFile(file)
  }
  target.value = ''
}

function handleDrop(event: DragEvent) {
  const file = event.dataTransfer?.files?.[0]
  if (file) {
    processFile(file)
  }
}

async function processFile(file: File) {
  if (!file.name.endsWith('.csv')) {
    ElMessage.error('请上传 CSV 格式的文件')
    return
  }

  try {
    const data = await parseCSV(file)
    rawCsvData.value = data
    store.setCsvData(data)

    // 自动检测列映射
    const mapping = autoDetectColumns(data[0] || [])
    store.setColumnMapping(mapping)

    // 如果检测到姓名列，自动应用映射
    if (mapping.name) {
      applyMapping()
    } else {
      ElMessage.warning('未能自动识别姓名列，请手动配置')
    }

    ElMessage.success(`解析成功，共 ${data.length - 1} 行数据`)
  } catch (error: any) {
    ElMessage.error(`解析失败：${error}`)
  }
}

function autoMap() {
  const mapping = autoDetectColumns(csvHeaders.value)
  store.setColumnMapping(mapping)
  if (mapping.name) {
    applyMapping()
  }
}

function applyMapping() {
  if (!columnMapping.value.name) return

  try {
    const participants = transformToParticipants(rawCsvData.value, columnMapping.value)
    store.setParticipants(participants)
  } catch (error: any) {
    ElMessage.error(error.message)
  }
}

function removeParticipant(index: number) {
  const newParticipants = [...participants.value]
  newParticipants.splice(index, 1)
  store.setParticipants(newParticipants)
}

function saveData() {
  store.saveToLocal()
  ElMessage.success('数据已保存')
}

function clearData() {
  store.setParticipants([])
  store.setCsvData([])
  rawCsvData.value = []
  duplicateResult.value = null
}

// 检查重复数据（工号和姓名）
function checkDuplicates() {
  const idMap = new Map<string, number>()
  const nameMap = new Map<string, number>()
  const duplicateIds: string[] = []
  const duplicateNames: string[] = []

  for (const p of participants.value) {
    // 检查工号重复
    if (p.id) {
      const count = idMap.get(p.id) || 0
      idMap.set(p.id, count + 1)
    }
    // 检查姓名重复
    if (p.name) {
      const count = nameMap.get(p.name) || 0
      nameMap.set(p.name, count + 1)
    }
  }

  // 收集重复的工号
  for (const [id, count] of idMap) {
    if (count > 1) {
      duplicateIds.push(`${id}(${count}次)`)
    }
  }

  // 收集重复的姓名
  for (const [name, count] of nameMap) {
    if (count > 1) {
      duplicateNames.push(`${name}(${count}次)`)
    }
  }

  duplicateResult.value = {
    hasDuplicate: duplicateIds.length > 0 || duplicateNames.length > 0,
    duplicateIds,
    duplicateNames,
  }

  if (duplicateResult.value.hasDuplicate) {
    ElMessage.warning('发现重复数据，请检查')
  } else {
    ElMessage.success('检查完成，无重复数据')
  }
}

// 移除重复项，只保留第一次出现的记录
function removeDuplicates() {
  const seenIds = new Set<string>()
  const seenNames = new Set<string>()
  const uniqueParticipants = []
  let removedCount = 0

  for (const p of participants.value) {
    const idKey = p.id || ''
    const nameKey = p.name || ''

    // 检查是否重复（工号或姓名已存在）
    const isDuplicateId = idKey && seenIds.has(idKey)
    const isDuplicateName = nameKey && seenNames.has(nameKey)

    if (isDuplicateId || isDuplicateName) {
      removedCount++
      continue
    }

    // 记录已出现的工号和姓名
    if (idKey) seenIds.add(idKey)
    if (nameKey) seenNames.add(nameKey)

    uniqueParticipants.push(p)
  }

  store.setParticipants(uniqueParticipants)
  duplicateResult.value = null

  ElMessage.success(`已移除 ${removedCount} 条重复数据，剩余 ${uniqueParticipants.length} 人`)
}

// 示例 CSV 数据
const sampleData: string[][] = [
  ['工号', '姓名', '手机号', '部门', '权重'],
  ['1001', '张三', '13800138001', '技术部', '10'],
  ['1002', '李四', '13800138002', '技术部', '8'],
  ['1003', '王五', '13800138003', '市场部', '10'],
  ['1004', '赵六', '13800138004', '市场部', '6'],
  ['1005', '钱七', '13800138005', '技术部', '9'],
  ['1006', '孙八', '13800138006', '财务部', '10'],
  ['1007', '周九', '13800138007', '财务部', '7'],
  ['1008', '吴十', '13800138008', '人事部', '10'],
  ['1009', '郑十一', '13800138009', '技术部', '8'],
  ['1010', '王十二', '13800138010', '市场部', '9'],
  ['1011', '刘十三', '13800138011', '人事部', '6'],
  ['1012', '陈十四', '13800138012', '技术部', '10'],
  ['1013', '杨十五', '13800138013', '市场部', '8'],
  ['1014', '黄十六', '13800138014', '财务部', '9'],
  ['1015', '赵十七', '13800138015', '技术部', '10'],
  ['1016', '吴十八', '13800138016', '人事部', '7'],
  ['1017', '周十九', '13800138017', '市场部', '10'],
  ['1018', '徐二十', '13800138018', '技术部', '8'],
  ['1019', '孙二一', '13800138019', '财务部', '9'],
  ['1020', '马二二', '13800138020', '市场部', '6'],
  ['1021', '朱二三', '13800138021', '技术部', '10'],
  ['1022', '胡二四', '13800138022', '人事部', '8'],
  ['1023', '郭二五', '13800138023', '市场部', '9'],
  ['1024', '林二六', '13800138024', '财务部', '7'],
  ['1025', '何二七', '13800138025', '技术部', '10'],
]

// 示例数据的列映射
const sampleMapping = {
  name: '姓名',
  id: '工号',
  phone: '手机号',
  department: '部门',
  weight: '权重',
}

// 静默加载示例数据（组件初始化时使用，不显示提示）
function loadSampleDataSilent() {
  rawCsvData.value = sampleData
  store.setCsvData(sampleData)
  store.setColumnMapping(sampleMapping)
  applyMapping()
}

// 手动加载示例数据（用户点击按钮时使用，显示提示）
function loadSampleData() {
  loadSampleDataSilent()
  ElMessage.success(`示例数据加载成功，共 ${sampleData.length - 1} 人`)
}
</script>

<style scoped>
.csv-uploader {
  padding: 24px;
}

/* 上传区域 */
.upload-area {
  margin-bottom: 32px;
}

.upload-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 48px 32px;
  background: rgba(255, 255, 255, 0.03);
  border: 2px dashed rgba(124, 58, 237, 0.4);
  border-radius: 24px;
  transition: all 0.3s;
  cursor: pointer;
}

.upload-content:hover,
.upload-content.drag-over {
  background: rgba(124, 58, 237, 0.1);
  border-color: rgba(124, 58, 237, 0.8);
  box-shadow: 0 0 30px rgba(124, 58, 237, 0.3);
}

.upload-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 80px;
  height: 80px;
  margin: 0 auto 20px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border-radius: 50%;
  color: #A78BFA;
  animation: float 3s ease-in-out infinite;
  box-shadow: 0 0 30px rgba(124, 58, 237, 0.3);
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.upload-text {
  font-size: 18px;
  font-weight: 600;
  color: #e2e8f0;
  margin: 0 0 8px 0;
}

.upload-subtext {
  font-size: 14px;
  color: #9ca3af;
  margin: 0 0 12px 0;
}

.upload-link {
  color: #A78BFA;
  cursor: pointer;
  text-decoration: underline;
}

.upload-link:hover {
  color: #F43F5E;
}

.upload-tip {
  font-size: 12px;
  color: #6b7280;
  margin: 0;
}

.hidden-input {
  display: none;
}

/* 映射配置 */
.mapping-config {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 16px;
  padding: 24px;
  margin-bottom: 32px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.section-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #e2e8f0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.auto-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border: 1px solid rgba(124, 58, 237, 0.4);
  border-radius: 20px;
  color: #A78BFA;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.auto-btn:hover {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.3) 0%, rgba(244, 63, 94, 0.3) 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.4), 0 0 30px rgba(244, 63, 94, 0.2);
}

.mapping-fields {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
}

.mapping-field {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.field-label {
  font-size: 13px;
  font-weight: 500;
  color: #9ca3af;
}

.field-label.required::after {
  content: ' *';
  color: #ef4444;
}

.field-select {
  padding: 12px 16px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  color: #e2e8f0;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%239ca3af' d='M6 9L1 4h2l3 5 3-5h2L6 9z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 16px center;
}

.field-select:hover {
  border-color: rgba(124, 58, 237, 0.5);
  background: rgba(255, 255, 255, 0.08);
}

.field-select:focus {
  outline: none;
  border-color: #7C3AED;
  box-shadow: 0 0 0 3px rgba(124, 58, 237, 0.3), 0 0 15px rgba(124, 58, 237, 0.2);
}

/* 预览区域 */
.preview-section {
  margin-top: 32px;
}

/* 重复检查提示 */
.duplicate-alert {
  margin-bottom: 20px;
  border-radius: 12px;
  overflow: hidden;
}

.duplicate-alert.warning {
  background: rgba(245, 158, 11, 0.1);
  border: 1px solid rgba(245, 158, 11, 0.3);
}

.duplicate-alert.success {
  background: rgba(34, 197, 94, 0.1);
  border: 1px solid rgba(34, 197, 94, 0.3);
}

.alert-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
}

.duplicate-alert.warning .alert-header {
  background: rgba(245, 158, 11, 0.1);
}

.duplicate-alert.success .alert-header {
  background: rgba(34, 197, 94, 0.1);
}

.alert-icon {
  color: inherit;
  flex-shrink: 0;
}

.alert-title {
  flex: 1;
  font-size: 14px;
  font-weight: 600;
  color: #e2e8f0;
}

.remove-duplicate-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 12px;
  background: rgba(239, 68, 68, 0.2);
  border: 1px solid rgba(239, 68, 68, 0.4);
  border-radius: 16px;
  color: #ef4444;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  margin-right: 8px;
}

.remove-duplicate-btn:hover {
  background: rgba(239, 68, 68, 0.3);
  transform: translateY(-1px);
}

.alert-close {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: #9ca3af;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.alert-close:hover {
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
}

.alert-body {
  padding: 12px 16px;
  border-top: 1px solid rgba(245, 158, 11, 0.2);
}

.duplicate-group {
  margin-bottom: 8px;
}

.duplicate-group:last-child {
  margin-bottom: 0;
}

.duplicate-label {
  font-size: 13px;
  font-weight: 500;
  color: #f59e0b;
  margin-right: 8px;
}

.duplicate-items {
  font-size: 13px;
  color: #e2e8f0;
}

.actions {
  display: flex;
  gap: 12px;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border: 1px solid rgba(124, 58, 237, 0.4);
  border-radius: 20px;
  color: #A78BFA;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.action-btn:hover {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.3) 0%, rgba(244, 63, 94, 0.3) 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.4), 0 0 30px rgba(244, 63, 94, 0.2);
}

.action-btn.danger {
  background: rgba(239, 68, 68, 0.2);
  border-color: rgba(239, 68, 68, 0.4);
  color: #ef4444;
}

.action-btn.danger:hover {
  background: rgba(239, 68, 68, 0.3);
}

/* 参与者卡片 */
.participant-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
  max-height: 500px;
  overflow-y: auto;
  padding: 4px;
}

.participant-card {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  overflow: hidden;
  transition: all 0.3s;
}

.participant-card:hover {
  border-color: rgba(124, 58, 237, 0.4);
  background: rgba(124, 58, 237, 0.05);
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(124, 58, 237, 0.3), 0 0 40px rgba(244, 63, 94, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.1) 0%, rgba(244, 63, 94, 0.1) 100%);
}

.participant-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: linear-gradient(135deg, #7C3AED 0%, #F43F5E 100%);
  color: white;
  font-size: 18px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 15px rgba(124, 58, 237, 0.4);
}

.remove-btn {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: rgba(239, 68, 68, 0.2);
  border: none;
  color: #ef4444;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.remove-btn:hover {
  background: rgba(239, 68, 68, 0.3);
  transform: scale(1.1);
}

.card-body {
  padding: 16px;
}

.participant-name {
  font-size: 16px;
  font-weight: 600;
  color: #e2e8f0;
  margin-bottom: 8px;
}

.participant-info {
  font-size: 13px;
  color: #6b7280;
  margin-bottom: 4px;
}

.load-more {
  text-align: center;
  margin-top: 20px;
}

.more-btn {
  padding: 10px 24px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  color: #9ca3af;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.more-btn:hover {
  background: rgba(124, 58, 237, 0.1);
  border-color: rgba(124, 58, 237, 0.4);
  color: #A78BFA;
  box-shadow: 0 4px 15px rgba(124, 58, 237, 0.3);
}

/* 滚动条样式 */
.participant-grid::-webkit-scrollbar {
  width: 8px;
}

.participant-grid::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
}

.participant-grid::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 4px;
}

.participant-grid::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.3);
}

@media (max-width: 768px) {
  .mapping-fields {
    grid-template-columns: 1fr;
  }

  .participant-grid {
    grid-template-columns: 1fr;
  }
}
</style>
