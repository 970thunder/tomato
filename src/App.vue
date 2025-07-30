<template>
  <div class="tomato-timer" :class="{ 'timer-active': isRunning || isPaused }">
    <!-- 庆祝弹窗 -->
    <div v-if="showCelebration" class="celebration-overlay">
      <!-- 彩带效果 -->
      <div class="confetti-container">
        <div v-for="i in 50" :key="i" class="confetti" :style="getConfettiStyle(i)"></div>
      </div>

      <!-- 庆祝卡片 -->
      <div class="celebration-card">
        <div class="celebration-header">
          <h2>🎉 专注完成！</h2>
          <div class="close-btn" @click="closeCelebration" @mouseenter="showCloseBtn = true"
            @mouseleave="showCloseBtn = false">
            <span v-if="showCloseBtn">×</span>
          </div>
        </div>

        <div class="celebration-content">
          <div class="tomato-success">🍅</div>
          <div class="focus-time">
            <span class="time-label">专注时间</span>
            <span class="time-value">{{ formatTime(selectedTime) }}</span>
          </div>
          <div class="success-message">
            恭喜你完成了这次专注！<br>
            休息一下，准备下一次挑战吧！
          </div>
        </div>

        <div class="celebration-actions">
          <button class="btn success" @click="restartTimer">
            再来一次
          </button>
          <button class="btn secondary" @click="closeCelebration">
            返回首页
          </button>
        </div>
      </div>
    </div>

    <!-- 大番茄时钟（仅在计时时显示） -->
    <div v-if="isRunning || isPaused" class="big-tomato-clock">
      <div class="big-tomato-icon">
        <svg class="big-progress-ring" :width="bigRingSize" :height="bigRingSize">
          <circle class="bg" :cx="bigRingSize / 2" :cy="bigRingSize / 2" :r="bigRingRadius" />
          <circle class="progress" :cx="bigRingSize / 2" :cy="bigRingSize / 2" :r="bigRingRadius"
            :stroke-dasharray="bigCircumference" :stroke-dashoffset="bigProgressOffset" />
        </svg>
        <div class="big-tomato-emoji">🍅</div>
      </div>

      <!-- 大时间显示 -->
      <div class="big-timer-display" :class="{ running: isRunning }">
        {{ formatTime(timeLeft) }}
      </div>

      <!-- 大状态文本 -->
      <div class="big-status-text">
        {{ statusText }}
      </div>
    </div>

    <!-- 原始界面（仅在未开始计时时显示） -->
    <div v-if="!isRunning && !isPaused" class="initial-interface">
      <!-- 番茄图标 -->
      <div class="tomato-icon">
        <svg class="progress-ring" :width="normalRingSize" :height="normalRingSize">
          <circle class="bg" :cx="normalRingSize / 2" :cy="normalRingSize / 2" :r="normalRingRadius" />
          <circle class="progress" :cx="normalRingSize / 2" :cy="normalRingSize / 2" :r="normalRingRadius"
            :stroke-dasharray="normalCircumference" :stroke-dashoffset="progressOffset" />
        </svg>
      </div>

      <!-- 时间显示 -->
      <div class="timer-display">
        {{ formatTime(timeLeft) }}
      </div>

      <!-- 状态文本 -->
      <div class="status-text">
        {{ statusText }}
      </div>

      <!-- 时间选择按钮 -->
      <div class="time-buttons">
        <button v-for="time in timeOptions" :key="time.value" class="time-btn"
          :class="{ active: selectedTime === time.value }" @click="selectTime(time.value)">
          {{ time.label }}
        </button>
      </div>

      <!-- 自定义时间输入 -->
      <div class="custom-time">
        <label>自定义时间:</label>
        <input v-model="customMinutes" type="number" min="1" max="180" placeholder="分钟"
          @keyup.enter="startCustomTimer" />
        <button class="btn secondary" @click="startCustomTimer">
          开始
        </button>
      </div>

      <!-- 开始按钮 -->
      <div class="control-buttons">
        <button v-if="timeLeft > 0" class="btn success start-btn" @click="startTimer">
          开始专注
        </button>
      </div>
    </div>

    <!-- 底部控制按钮（仅在计时时显示） -->
    <div v-if="isRunning || isPaused" class="bottom-controls">
      <div class="control-buttons">
        <button v-if="isRunning" class="btn secondary" @click="pauseTimer">
          暂停
        </button>
        <button v-if="isPaused" class="btn success" @click="resumeTimer">
          继续
        </button>
        <button class="btn danger" @click="resetTimer">
          重置
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onUnmounted, onMounted } from 'vue'

export default {
  name: 'TomatoTimer',
  setup() {
    const timeLeft = ref(0)
    const isRunning = ref(false)
    const isPaused = ref(false)
    const selectedTime = ref(0)
    const customMinutes = ref('')
    const showCelebration = ref(false)
    const showCloseBtn = ref(false)
    const windowWidth = ref(window.innerWidth)
    let timer = null

    const timeOptions = [
      { label: '15分钟', value: 15 * 60 },
      { label: '30分钟', value: 30 * 60 },
      { label: '1小时', value: 60 * 60 }
    ]

    // 响应式圆环尺寸
    const normalRingSize = computed(() => {
      if (windowWidth.value <= 360) return 80
      if (windowWidth.value <= 480) return 90
      if (windowWidth.value <= 768) return 100
      return 120
    })

    const bigRingSize = computed(() => {
      if (windowWidth.value <= 360) return 120
      if (windowWidth.value <= 480) return 140
      if (windowWidth.value <= 768) return 160
      return 200
    })

    const normalRingRadius = computed(() => {
      return normalRingSize.value * 0.375 // 45/120 的比例
    })

    const bigRingRadius = computed(() => {
      return bigRingSize.value * 0.4 // 80/200 的比例
    })

    const normalCircumference = computed(() => {
      return 2 * Math.PI * normalRingRadius.value
    })

    const bigCircumference = computed(() => {
      return 2 * Math.PI * bigRingRadius.value
    })

    const statusText = computed(() => {
      if (timeLeft.value === 0) return '选择时间开始专注'
      if (isRunning.value) return '专注中...'
      if (isPaused.value) return '已暂停'
      return '准备开始'
    })

    const progressOffset = computed(() => {
      if (selectedTime.value === 0 || timeLeft.value === 0) {
        // 没有选择时间时，或倒计时结束时
        return selectedTime.value === 0 ? normalCircumference.value : 0
      }
      // 计算剩余进度：剩余时间越多，offset越大（显示的圆环越少）
      const remainingRatio = timeLeft.value / selectedTime.value
      const offset = remainingRatio * normalCircumference.value
      return offset
    })

    const bigProgressOffset = computed(() => {
      if (selectedTime.value === 0 || timeLeft.value === 0) {
        // 没有选择时间时，或倒计时结束时
        return selectedTime.value === 0 ? bigCircumference.value : 0
      }
      // 计算剩余进度：剩余时间越多，offset越大（显示的圆环越少）
      const remainingRatio = timeLeft.value / selectedTime.value
      const offset = remainingRatio * bigCircumference.value
      return offset
    })

    const formatTime = (seconds) => {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
    }

    const selectTime = (seconds) => {
      if (isRunning.value || isPaused.value) {
        resetTimer()
      }
      selectedTime.value = seconds
      timeLeft.value = seconds
    }

    const startCustomTimer = () => {
      const minutes = parseInt(customMinutes.value)
      if (minutes && minutes > 0 && minutes <= 180) {
        const seconds = minutes * 60
        selectTime(seconds)
        startTimer()
        customMinutes.value = ''
      }
    }

    const startTimer = () => {
      if (timeLeft.value > 0) {
        isRunning.value = true
        isPaused.value = false
        timer = setInterval(() => {
          timeLeft.value--
          if (timeLeft.value <= 0) {
            clearInterval(timer)
            isRunning.value = false
            isPaused.value = false
            // 显示庆祝弹窗
            showCelebration.value = true
            // 播放提示音
            playNotification()
          }
        }, 1000)
      }
    }

    const pauseTimer = () => {
      if (isRunning.value) {
        clearInterval(timer)
        isRunning.value = false
        isPaused.value = true
      }
    }

    const resumeTimer = () => {
      if (isPaused.value) {
        startTimer()
      }
    }

    const resetTimer = () => {
      clearInterval(timer)
      isRunning.value = false
      isPaused.value = false
      timeLeft.value = selectedTime.value
    }

    const closeCelebration = () => {
      showCelebration.value = false
      showCloseBtn.value = false
      resetTimer()
    }

    const restartTimer = () => {
      showCelebration.value = false
      showCloseBtn.value = false
      timeLeft.value = selectedTime.value
      startTimer()
    }

    const getConfettiStyle = (index) => {
      const colors = ['#ff6b6b', '#4ecdc4', '#45b7d1', '#96ceb4', '#feca57', '#ff9ff3', '#54a0ff', '#5f27cd']
      const color = colors[index % colors.length]
      const left = Math.random() * 100
      const animationDelay = Math.random() * 3
      const animationDuration = 3 + Math.random() * 2

      return {
        left: `${left}%`,
        backgroundColor: color,
        animationDelay: `${animationDelay}s`,
        animationDuration: `${animationDuration}s`
      }
    }

    const playNotification = () => {
      // 创建音频上下文播放提示音
      const audioContext = new (window.AudioContext || window.webkitAudioContext)()
      const oscillator = audioContext.createOscillator()
      const gainNode = audioContext.createGain()

      oscillator.connect(gainNode)
      gainNode.connect(audioContext.destination)

      oscillator.frequency.setValueAtTime(800, audioContext.currentTime)
      oscillator.frequency.setValueAtTime(600, audioContext.currentTime + 0.1)
      oscillator.frequency.setValueAtTime(800, audioContext.currentTime + 0.2)

      gainNode.gain.setValueAtTime(0.3, audioContext.currentTime)
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3)

      oscillator.start(audioContext.currentTime)
      oscillator.stop(audioContext.currentTime + 0.3)

      // 显示通知
      if ('Notification' in window && Notification.permission === 'granted') {
        new Notification('番茄钟完成！', {
          body: '专注时间结束，休息一下吧！',
          icon: '/tomato.svg'
        })
      }
    }

    const handleResize = () => {
      windowWidth.value = window.innerWidth
    }

    // 请求通知权限
    if ('Notification' in window && Notification.permission === 'default') {
      Notification.requestPermission()
    }

    // 组件挂载时添加窗口尺寸监听
    onMounted(() => {
      window.addEventListener('resize', handleResize)
    })

    // 组件卸载时清理定时器和事件监听
    onUnmounted(() => {
      if (timer) {
        clearInterval(timer)
      }
      window.removeEventListener('resize', handleResize)
    })

    return {
      timeLeft,
      isRunning,
      isPaused,
      selectedTime,
      customMinutes,
      timeOptions,
      statusText,
      progressOffset,
      bigProgressOffset,
      showCelebration,
      showCloseBtn,
      normalRingSize,
      bigRingSize,
      normalRingRadius,
      bigRingRadius,
      normalCircumference,
      bigCircumference,
      formatTime,
      selectTime,
      startCustomTimer,
      startTimer,
      pauseTimer,
      resumeTimer,
      resetTimer,
      closeCelebration,
      restartTimer,
      getConfettiStyle
    }
  }
}
</script>