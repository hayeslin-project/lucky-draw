<template>
  <div class="settings-panel">
    <div class="panel-header">
      <h2>
        <Icon name="settings" size="lg" />
        <span>抽奖设置</span>
      </h2>
    </div>

    <!-- 抽奖模式 -->
    <div class="setting-section">
      <label class="section-label">抽奖模式</label>
      <div class="mode-selector">
        <button
          v-for="mode in modes"
          :key="mode.value"
          :class="['mode-btn', { active: config.mode === mode.value }]"
          @click="updateMode(mode.value)"
        >
          <Icon :name="mode.icon" size="xl" />
          <span>{{ mode.name }}</span>
        </button>
      </div>
    </div>

    <!-- 奖项设置 -->
    <div class="setting-section">
      <div class="section-header">
        <label class="section-label">奖项设置</label>
        <button class="add-btn" @click="addPrize">
          <Icon name="plus" size="sm" />
          <span>添加奖项</span>
        </button>
      </div>

      <div class="prizes-container">
        <div
          v-for="prize in config.prizes"
          :key="prize.id"
          :class="['prize-card', { active: currentPrizeId === prize.id }]"
          :style="currentPrizeId === prize.id ? { borderColor: prize.color, boxShadow: `0 0 20px ${prize.color}40` } : {}"
          @click="selectPrize(prize.id)"
        >
          <div class="prize-header">
            <div class="prize-name">{{ prize.name }}</div>
            <button
              v-if="config.prizes.length > 1"
              class="delete-btn"
              @click.stop="removePrize(prize.id)"
            >
              <Icon name="x" size="sm" />
            </button>
          </div>
          <div class="prize-count">
            <div class="count-input">
              <button class="count-btn" @click.stop="updatePrizeCount(prize.id, -1)">
                <Icon name="minus" size="sm" />
              </button>
              <span class="count-number">{{ prize.count }}</span>
              <button class="count-btn" @click.stop="updatePrizeCount(prize.id, 1)">
                <Icon name="plus" size="sm" />
              </button>
            </div>
            <span class="count-label">人</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 分组设置 -->
    <div v-if="config.mode === 'group'" class="setting-section">
      <label class="section-label">分组设置</label>
      <div class="group-settings">
        <div
          v-for="dept in departments"
          :key="dept"
          class="group-item"
        >
          <label class="group-checkbox">
            <input type="checkbox" v-model="groupSettingsEnabled[dept]" />
            <span class="checkbox-custom"></span>
            <span class="group-name">{{ dept }}</span>
            <span class="group-count">({{ deptParticipantCount(dept) }}人)</span>
          </label>
          <div v-if="groupSettingsEnabled[dept]" class="group-prizes">
            <div v-for="prize in config.prizes" :key="prize.id" class="group-prize-row">
              <span class="prize-label">{{ prize.name }}</span>
              <div class="count-input small">
                <button class="count-btn" @click="adjustGroupCount(dept, prize.id, -1)">
                  <Icon name="minus" size="sm" />
                </button>
                <span class="count-number">{{ getGroupCount(dept, prize.id) }}</span>
                <button class="count-btn" @click="adjustGroupCount(dept, prize.id, 1)">
                  <Icon name="plus" size="sm" />
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 其他设置 -->
    <div class="setting-section">
      <label class="section-label">其他选项</label>
      <div class="options-container">
        <label class="option-switch">
          <input type="checkbox" v-model="config.allowRepeat" />
          <span class="switch-slider"></span>
          <span class="switch-label">允许重复中奖</span>
        </label>
        <label class="option-switch">
          <input type="checkbox" v-model="config.enableVoice" />
          <span class="switch-slider"></span>
          <span class="switch-label">语音播报</span>
        </label>
      </div>
    </div>

    <!-- 统计信息 -->
    <div class="stats-section">
      <div class="stat-card">
        <div class="stat-icon">
          <Icon name="users" size="lg" />
        </div>
        <div class="stat-value">{{ stats.total }}</div>
        <div class="stat-label">参与人数</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon">
          <Icon name="trophy" size="lg" />
        </div>
        <div class="stat-value">{{ stats.winnerCount }}</div>
        <div class="stat-label">已中奖</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon">
          <Icon name="user" size="lg" />
        </div>
        <div class="stat-value">{{ stats.remainingCount }}</div>
        <div class="stat-label">剩余人数</div>
      </div>
    </div>

    <!-- 奖项中奖统计 -->
    <div class="prize-stats">
      <div
        v-for="prize in config.prizes"
        :key="prize.id"
        class="prize-stat-item"
      >
        <div class="prize-stat-name" :style="{ color: prize.color }">{{ prize.name }}</div>
        <div class="prize-stat-bar">
          <div
            class="prize-stat-fill"
            :style="{
              width: `${Math.min((prizeStats[prize.id] || 0) / prize.count * 100, 100)}%`,
              background: prize.color
            }"
          ></div>
        </div>
        <div class="prize-stat-count">{{ prizeStats[prize.id] || 0 }} / {{ prize.count }}</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, watch } from 'vue'
import { useLotteryStore } from '@/stores/lottery'
import type { LotteryMode } from '@/utils/lottery'
import Icon from '@/components/common/Icon.vue'

const store = useLotteryStore()

const config = computed(() => store.config)
const currentPrizeId = computed({
  get: () => store.currentPrizeId,
  set: (val) => store.setCurrentPrize(val),
})
const departments = computed(() => store.departments)
const stats = computed(() => store.stats)
const prizeStats = computed(() => store.prizeStats)

const modes = [
  { value: 'normal' as LotteryMode, name: '普通随机', icon: 'dice' },
  { value: 'weighted' as LotteryMode, name: '权重抽奖', icon: 'scale' },
  { value: 'group' as LotteryMode, name: '分组抽奖', icon: 'users' },
]

// 分组设置（从 store 获取）
const groupSettingsEnabled = computed(() => store.groupSettingsEnabled)
const groupCounts = computed(() => store.groupCounts)

// 监听部门变化，初始化分组设置
watch(
  departments,
  (newDepts) => {
    newDepts.forEach((dept) => {
      if (groupSettingsEnabled.value[dept] === undefined) {
        groupSettingsEnabled.value[dept] = false
      }

      if (!groupCounts.value[dept]) {
        groupCounts.value[dept] = {}
      }
      config.value.prizes.forEach((prize) => {
        if (groupCounts.value[dept][prize.id] === undefined) {
          groupCounts.value[dept][prize.id] = 0
        }
      })
    })
  },
  { immediate: true }
)

// 获取部门参与人数
function deptParticipantCount(dept: string) {
  return store.participants.filter((p) => p.department === dept).length
}

// 获取分组计数
function getGroupCount(dept: string, prizeId: string) {
  return groupCounts.value[dept]?.[prizeId] || 0
}

// 设置分组计数
function setGroupCount(dept: string, prizeId: string, value: number) {
  if (!store.groupCounts[dept]) {
    store.groupCounts[dept] = {}
  }
  store.groupCounts[dept][prizeId] = value
}

// 调整分组计数
function adjustGroupCount(dept: string, prizeId: string, delta: number) {
  const current = getGroupCount(dept, prizeId)
  const newValue = Math.max(0, Math.min(deptParticipantCount(dept), current + delta))
  setGroupCount(dept, prizeId, newValue)
}

function updateMode(mode: LotteryMode) {
  store.updateConfig({ mode })
}

function updatePrizeCount(prizeId: string, delta: number) {
  const prize = config.value.prizes.find((p) => p.id === prizeId)
  if (prize) {
    const newCount = Math.max(1, prize.count + delta)
    store.updatePrize(prizeId, { count: newCount })
  }
}

function addPrize() {
  const id = `prize${Date.now()}`
  const colors = ['#f56c6c', '#e6a23c', '#409eff', '#67c23a', '#909399', '#f093fb', '#4facfe']
  const color = colors[config.value.prizes.length % colors.length]

  store.addPrize({
    id,
    name: `奖项${config.value.prizes.length + 1}`,
    count: 1,
    color,
  })
}

function selectPrize(prizeId: string) {
  store.setCurrentPrize(prizeId)
}

function removePrize(prizeId: string) {
  store.removePrize(prizeId)
  if (currentPrizeId.value === prizeId) {
    const firstPrize = config.value.prizes[0]
    if (firstPrize) {
      store.setCurrentPrize(firstPrize.id)
    }
  }
}
</script>

<style scoped>
.settings-panel {
  padding: 24px;
}

.panel-header {
  margin-bottom: 32px;
}

.panel-header h2 {
  margin: 0;
  font-size: 24px;
  font-weight: 700;
  background: linear-gradient(135deg, #7C3AED 0%, #A78BFA 50%, #F43F5E 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  display: flex;
  align-items: center;
  gap: 12px;
  text-shadow: 0 0 30px rgba(124, 58, 237, 0.5);
}

.setting-section {
  margin-bottom: 32px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.section-label {
  display: block;
  margin-bottom: 12px;
  font-size: 14px;
  font-weight: 600;
  color: #9ca3af;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.add-btn {
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

.add-btn:hover {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.3) 0%, rgba(244, 63, 94, 0.3) 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.4), 0 0 30px rgba(244, 63, 94, 0.2);
}

/* 模式选择器 */
.mode-selector {
  display: flex;
  gap: 12px;
}

.mode-btn {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.03);
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
}

.mode-btn:hover {
  background: rgba(124, 58, 237, 0.1);
  border-color: rgba(124, 58, 237, 0.5);
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.2);
}

.mode-btn.active {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border-color: #7C3AED;
  color: #A78BFA;
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.4), 0 0 30px rgba(244, 63, 94, 0.2);
}

/* 奖项卡片 */
.prizes-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 16px;
}

.prize-card {
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.03);
  cursor: pointer;
  transition: all 0.3s;
}

.prize-card:hover {
  border-color: rgba(124, 58, 237, 0.5);
  background: rgba(124, 58, 237, 0.05);
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(124, 58, 237, 0.3), 0 0 40px rgba(244, 63, 94, 0.1);
}

.prize-card.active {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.1) 0%, rgba(244, 63, 94, 0.1) 100%);
}

.prize-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 16px;
}

.prize-name {
  font-size: 16px;
  font-weight: 600;
  color: #e2e8f0;
}

.delete-btn {
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

.delete-btn:hover {
  background: rgba(239, 68, 68, 0.3);
  transform: scale(1.1);
}

.prize-count {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.count-input {
  display: flex;
  align-items: center;
  gap: 8px;
}

.count-input.small {
  transform: scale(0.85);
}

.count-btn {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: #e2e8f0;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.count-btn:hover {
  background: rgba(124, 58, 237, 0.3);
  color: #A78BFA;
  box-shadow: 0 0 15px rgba(124, 58, 237, 0.4);
}

.count-number {
  min-width: 32px;
  text-align: center;
  font-size: 24px;
  font-weight: 700;
  color: #e2e8f0;
}

.count-label {
  font-size: 12px;
  color: #6b7280;
}

/* 分组设置 */
.group-settings {
  max-height: 300px;
  overflow-y: auto;
  padding-right: 8px;
}

.group-item {
  margin-bottom: 12px;
}

.group-checkbox {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s;
}

.group-checkbox:hover {
  background: rgba(255, 255, 255, 0.05);
}

.group-checkbox input[type="checkbox"] {
  display: none;
}

.checkbox-custom {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 6px;
  position: relative;
  transition: all 0.3s;
}

.group-checkbox input:checked + .checkbox-custom {
  background: linear-gradient(135deg, #7C3AED 0%, #F43F5E 100%);
  border-color: #7C3AED;
  box-shadow: 0 0 15px rgba(124, 58, 237, 0.5);
}

.group-checkbox input:checked + .checkbox-custom::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 12px;
  font-weight: 700;
}

.group-name {
  flex: 1;
  font-weight: 500;
  color: #e2e8f0;
}

.group-count {
  font-size: 12px;
  color: #6b7280;
}

.group-prizes {
  margin-top: 12px;
  padding: 0 0 0 44px;
}

.group-prize-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: rgba(255, 255, 255, 0.02);
  border-radius: 8px;
  margin-bottom: 8px;
}

.prize-label {
  font-size: 13px;
  color: #9ca3af;
}

/* 选项开关 */
.options-container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.option-switch {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s;
}

.option-switch:hover {
  background: rgba(255, 255, 255, 0.05);
}

.option-switch input[type="checkbox"] {
  display: none;
}

.switch-slider {
  width: 48px;
  height: 26px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 13px;
  position: relative;
  transition: all 0.3s;
}

.switch-slider::after {
  content: '';
  position: absolute;
  top: 3px;
  left: 3px;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  transition: all 0.3s;
}

.option-switch input:checked + .switch-slider {
  background: linear-gradient(135deg, #7C3AED 0%, #F43F5E 100%);
  box-shadow: 0 0 15px rgba(124, 58, 237, 0.5);
}

.option-switch input:checked + .switch-slider::after {
  left: 25px;
}

.switch-label {
  font-size: 14px;
  color: #e2e8f0;
  font-weight: 500;
}

/* 统计信息 */
.stats-section {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-top: 32px;
}

.stat-card {
  padding: 20px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.1) 0%, rgba(244, 63, 94, 0.1) 100%);
  border: 1px solid rgba(124, 58, 237, 0.3);
  border-radius: 16px;
  text-align: center;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.2);
}

.stat-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  margin: 0 auto 12px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border-radius: 12px;
  color: #A78BFA;
  box-shadow: 0 0 15px rgba(124, 58, 237, 0.3);
}

.stat-value {
  font-size: 32px;
  font-weight: 800;
  background: linear-gradient(135deg, #7C3AED 0%, #A78BFA 50%, #F43F5E 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 4px;
  text-shadow: 0 0 30px rgba(124, 58, 237, 0.5);
}

.stat-label {
  font-size: 12px;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 1px;
}

/* 奖项统计 */
.prize-stats {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.prize-stat-item {
  display: grid;
  grid-template-columns: 80px 1fr 60px;
  align-items: center;
  gap: 16px;
  padding: 12px 16px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 12px;
}

.prize-stat-name {
  font-size: 14px;
  font-weight: 600;
}

.prize-stat-bar {
  height: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  overflow: hidden;
}

.prize-stat-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.5s ease;
}

.prize-stat-count {
  font-size: 13px;
  color: #9ca3af;
  text-align: right;
  font-weight: 500;
}

/* 滚动条样式 */
.group-settings::-webkit-scrollbar {
  width: 6px;
}

.group-settings::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 3px;
}

.group-settings::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 3px;
}

.group-settings::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.3);
}

@media (max-width: 768px) {
  .prizes-container {
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  }

  .stats-section {
    grid-template-columns: 1fr;
  }

  .prize-stat-item {
    grid-template-columns: 1fr 60px;
    gap: 12px;
  }

  .prize-stat-bar {
    grid-column: 1;
  }
}
</style>
