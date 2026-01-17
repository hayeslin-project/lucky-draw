<template>
  <div class="lottery-board">
    <!-- 当前奖项信息 -->
    <div class="current-prize">
      <div class="prize-badge">
        <Icon name="party" size="md" />
        <span>{{ currentPrize?.name || '请选择奖项' }}</span>
      </div>
      <div class="prize-count">
        <span class="count-number">{{ currentPrize?.count || 0 }}</span>
        <span class="count-label">人</span>
      </div>
    </div>

    <!-- 抽奖展示区域 -->
    <div class="lottery-display">
      <!-- 等待开始状态 -->
      <div v-if="!isRunning && !isAnimating" class="waiting-state">
        <div class="waiting-icon">
          <Icon name="dice" size="2xl" />
        </div>
        <p class="waiting-text">点击"开始抽奖"</p>
        <p class="waiting-hint">准备开始新一期抽奖</p>
      </div>

      <!-- 滚动动画状态 -->
      <div v-else class="rolling-state">
        <div class="rolling-container">
          <div
            v-for="(name, index) in displayNames"
            :key="`${name}-${index}`"
            class="rolling-card"
            :style="getRollingStyle(index)"
          >
            <div class="card-inner">
              <div class="card-front">
                <div class="card-icon">
                  <Icon name="question" size="xl" />
                </div>
              </div>
              <div class="card-back">
                <span class="winner-name">{{ name }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 操作按钮 -->
    <div class="control-buttons">
      <template v-if="!isRunning">
        <button
          v-if="!isAnimating"
          class="btn btn-primary"
          :class="{ disabled: !canStart }"
          :disabled="!canStart"
          @click="startLottery"
        >
          <Icon name="play" size="md" />
          <span>开始抽奖</span>
        </button>
        <button
          v-else
          class="btn btn-danger"
          @click="stopLottery"
        >
          <Icon name="pause" size="md" />
          <span>停止</span>
        </button>
      </template>
      <div v-else class="confirm-actions">
        <button class="btn btn-secondary" @click="cancelLottery">
          <Icon name="x" size="md" />
          <span>取消</span>
        </button>
        <button
          class="btn btn-success"
          @click="confirmResult"
        >
          <Icon name="check" size="md" />
          <span>确认结果</span>
        </button>
      </div>
    </div>

    <!-- 分组抽奖按钮 -->
    <button
      v-if="config.mode === 'group'"
      class="group-lottery-btn"
      :class="{ disabled: !canStartGroupLottery }"
      :disabled="!canStartGroupLottery"
      @click="startGroupLottery"
    >
      <Icon name="users" size="md" />
      <span>分组抽奖</span>
    </button>

    <!-- 快速切换奖项 -->
    <div v-if="config.prizes.length > 1" class="prize-selector">
      <button
        v-for="prize in config.prizes"
        :key="prize.id"
        :class="['prize-tag', { active: currentPrizeId === prize.id }]"
        :style="currentPrizeId === prize.id ? { borderColor: prize.color, background: `${prize.color}20` } : {}"
        @click="selectPrize(prize.id)"
      >
        {{ prize.name }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onUnmounted } from 'vue'
import { ElMessage } from 'element-plus'
import { useLotteryStore } from '@/stores/lottery'
import { createAnimationGenerator } from '@/utils/lottery'
import Icon from '@/components/common/Icon.vue'

const store = useLotteryStore()
const emit = defineEmits(['result', 'groupResult'])

const currentPrize = computed(() => store.currentPrize)
const currentPrizeId = computed({
  get: () => store.currentPrizeId,
  set: (val) => store.setCurrentPrize(val),
})
const config = computed(() => store.config)
const isRunning = computed(() => store.isRunning)
const isAnimating = computed(() => store.isAnimating)
const participants = computed(() => store.participants)
const winnerIds = computed(() => store.winnerIds)

// 显示的名字列表（用于动画）
const displayNames = ref<string[]>([])
// 动画定时器
let animationTimer: number | null = null
// 临时抽奖结果
const tempWinners = ref<any[]>([])

// 是否可以开始抽奖
const canStart = computed(() => {
  if (!currentPrize.value || participants.value.length === 0) return false
  if (config.value.allowRepeat) return true
  return participants.value.length > winnerIds.value.size
})

// 是否可以开始分组抽奖
const canStartGroupLottery = computed(() => {
  return config.value.mode === 'group' && participants.value.length > 0
})

// 监听中奖名单变化，更新显示
watch(tempWinners, (winners) => {
  displayNames.value = winners.map((w) => w.name || w)
})

function startLottery() {
  if (!currentPrize.value) {
    ElMessage.warning('请先选择奖项')
    return
  }

  const count = currentPrize.value.count
  const excludeIds = config.value.allowRepeat ? new Set<string>() : winnerIds.value
  const candidates = participants.value.filter((p) => !excludeIds.has(p.id))

  if (candidates.length < count) {
    ElMessage.warning(`参与人数不足，还差 ${count - candidates.length} 人`)
    return
  }

  // 初始化显示
  displayNames.value = Array(count).fill('???')
  tempWinners.value = Array(count).fill({})

  // 设置动画状态
  store.isAnimating = true

  // 创建动画生成器
  const generator = createAnimationGenerator(candidates)

  // 开始滚动动画
  animationTimer = window.setInterval(() => {
    for (let i = 0; i < count; i++) {
      const participant = generator()
      displayNames.value[i] = participant.name
      tempWinners.value[i] = participant
    }
  }, 50)
}

function stopLottery() {
  if (animationTimer) {
    clearInterval(animationTimer)
    animationTimer = null
  }

  // 执行真正的抽奖
  const selected = store.drawLottery()

  if (selected.length > 0) {
    displayNames.value = selected.map((p) => p.name)
    tempWinners.value = selected
    store.isRunning = true
    store.isAnimating = false
  } else {
    ElMessage.error('抽奖失败，请重试')
    store.isAnimating = false
  }
}

function cancelLottery() {
  store.isRunning = false
  displayNames.value = []
  tempWinners.value = []
}

function confirmResult() {
  store.confirmWinners(tempWinners.value)
  emit('result', tempWinners.value)

  store.isRunning = false
  displayNames.value = []
  tempWinners.value = []
}

function startGroupLottery() {
  const groupSettings: Record<string, any> = {}

  // 从 store 获取分组设置
  const enabledDepts = Object.keys(store.groupSettingsEnabled).filter(
    (dept) => store.groupSettingsEnabled[dept]
  )

  if (enabledDepts.length === 0) {
    ElMessage.warning('请先选择要抽奖的分组')
    return
  }

  for (const dept of enabledDepts) {
    const deptCounts: any[] = []
    const prizeSettings = store.groupCounts[dept] || {}

    for (const prize of config.value.prizes) {
      const count = prizeSettings[prize.id] || 0
      if (count > 0) {
        deptCounts.push({
          prizeId: prize.id,
          count,
        })
      }
    }

    if (deptCounts.length > 0) {
      groupSettings[dept] = deptCounts
    }
  }

  if (Object.keys(groupSettings).length === 0) {
    ElMessage.warning('请设置各组抽取人数')
    return
  }

  const results = store.drawGroupLottery(groupSettings)
  emit('groupResult', results)
}

function selectPrize(prizeId: string) {
  store.setCurrentPrize(prizeId)
}

function getRollingStyle(index: number) {
  const delay = index * 0.05
  return {
    animationDelay: `${delay}s`,
  }
}

// 组件卸载时清理定时器
onUnmounted(() => {
  if (animationTimer) {
    clearInterval(animationTimer)
  }
})
</script>

<style scoped>
.lottery-board {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100%;
  position: relative;
}

/* 当前奖项 */
.current-prize {
  text-align: center;
  margin-bottom: 40px;
}

.prize-badge {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 12px 32px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.3) 0%, rgba(244, 63, 94, 0.3) 100%);
  border: 2px solid rgba(124, 58, 237, 0.5);
  border-radius: 50px;
  font-size: 20px;
  font-weight: 600;
  color: white;
  backdrop-filter: blur(10px);
  box-shadow: 0 0 30px rgba(124, 58, 237, 0.4), 0 0 60px rgba(244, 63, 94, 0.2);
  margin-bottom: 16px;
  animation: neonGlow 2s ease-in-out infinite alternate;
}

@keyframes neonGlow {
  from {
    box-shadow: 0 0 20px rgba(124, 58, 237, 0.4), 0 0 40px rgba(244, 63, 94, 0.2);
  }
  to {
    box-shadow: 0 0 40px rgba(124, 58, 237, 0.6), 0 0 80px rgba(244, 63, 94, 0.4);
  }
}

.prize-count {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.count-number {
  font-size: 56px;
  font-weight: 800;
  background: linear-gradient(135deg, #7C3AED 0%, #A78BFA 50%, #F43F5E 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  line-height: 1;
  text-shadow: 0 0 30px rgba(124, 58, 237, 0.5);
}

.count-label {
  font-size: 16px;
  color: #9ca3af;
}

/* 抽奖展示区域 */
.lottery-display {
  flex: 1;
  width: 100%;
  min-height: 350px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 40px;
}

.waiting-state {
  text-align: center;
  animation: fadeInUp 0.5s ease;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.waiting-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 120px;
  height: 120px;
  margin: 0 auto 24px;
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border-radius: 50%;
  color: #A78BFA;
  box-shadow: 0 0 30px rgba(124, 58, 237, 0.3), 0 0 60px rgba(244, 63, 94, 0.2);
  animation: floatGlow 3s ease-in-out infinite;
}

@keyframes floatGlow {
  0%, 100% {
    transform: translateY(0) rotate(0deg);
    box-shadow: 0 0 30px rgba(124, 58, 237, 0.3), 0 0 60px rgba(244, 63, 94, 0.2);
  }
  50% {
    transform: translateY(-15px) rotate(5deg);
    box-shadow: 0 0 50px rgba(124, 58, 237, 0.5), 0 0 100px rgba(244, 63, 94, 0.3);
  }
}

.waiting-text {
  font-size: 24px;
  font-weight: 600;
  color: #e2e8f0;
  margin: 0 0 8px 0;
}

.waiting-hint {
  font-size: 14px;
  color: #6b7280;
  margin: 0;
}

/* 滚动状态 */
.rolling-state {
  width: 100%;
}

.rolling-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 24px;
  padding: 20px;
}

.rolling-card {
  width: 200px;
  height: 140px;
  perspective: 1000px;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.rolling-card:hover .card-inner {
  transform: rotateY(10deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-front {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.2) 0%, rgba(244, 63, 94, 0.2) 100%);
  border: 2px solid rgba(124, 58, 237, 0.4);
  backdrop-filter: blur(10px);
  animation: cardPulse 1s ease-in-out infinite;
}

@keyframes cardPulse {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 0 20px rgba(124, 58, 237, 0.3);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 40px rgba(124, 58, 237, 0.5);
  }
}

.card-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255, 255, 255, 0.8);
}

.card-back {
  background: linear-gradient(135deg, #7C3AED 0%, #F43F5E 100%);
  transform: rotateY(180deg);
  box-shadow: 0 10px 40px rgba(124, 58, 237, 0.5), 0 0 60px rgba(244, 63, 94, 0.3);
}

.winner-name {
  font-size: 28px;
  font-weight: 700;
  color: white;
}

/* 控制按钮 */
.control-buttons {
  display: flex;
  gap: 16px;
  margin-bottom: 24px;
}

.btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 16px 40px;
  border: none;
  border-radius: 16px;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transition: left 0.5s;
}

.btn:hover::before {
  left: 100%;
}

.btn:hover:not(.disabled) {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.btn:active:not(.disabled) {
  transform: translateY(-1px);
}

.btn:disabled,
.btn.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary {
  background: linear-gradient(135deg, #7C3AED 0%, #F43F5E 100%);
  color: white;
  box-shadow: 0 4px 20px rgba(124, 58, 237, 0.4), 0 0 30px rgba(244, 63, 94, 0.2);
}

.btn-danger {
  background: linear-gradient(135deg, #F43F5E 0%, #FF6B6B 100%);
  color: white;
  box-shadow: 0 4px 20px rgba(244, 63, 94, 0.4);
}

.btn-secondary {
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: #e2e8f0;
}

.btn-secondary:hover {
  background: rgba(255, 255, 255, 0.15);
}

.btn-success {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  color: white;
  box-shadow: 0 4px 20px rgba(79, 172, 254, 0.4);
}

.confirm-actions {
  display: flex;
  gap: 16px;
}

/* 分组抽奖按钮 */
.group-lottery-btn {
  position: absolute;
  bottom: 0;
  right: 0;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  border: 2px solid rgba(102, 126, 234, 0.5);
  border-radius: 50px;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  backdrop-filter: blur(10px);
}

.group-lottery-btn:hover:not(.disabled) {
  background: rgba(102, 126, 234, 0.2);
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
}

.group-lottery-btn.disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* 奖项选择器 */
.prize-selector {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
  margin-top: 24px;
}

.prize-tag {
  padding: 10px 20px;
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 50px;
  background: rgba(255, 255, 255, 0.05);
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
}

.prize-tag:hover {
  border-color: rgba(102, 126, 234, 0.5);
  background: rgba(102, 126, 234, 0.1);
  transform: translateY(-2px);
}

.prize-tag.active {
  color: white;
  font-weight: 600;
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.3);
}

@media (max-width: 768px) {
  .rolling-card {
    width: 140px;
    height: 100px;
  }

  .winner-name {
    font-size: 20px;
  }

  .btn {
    padding: 14px 28px;
    font-size: 16px;
  }

  .count-number {
    font-size: 40px;
  }
}
</style>
