<template>
  <div class="tomato-timer" :class="{ 'timer-active': isRunning || isPaused }">
    <!-- 大番茄时钟（仅在计时时显示） -->
    <div v-if="isRunning || isPaused" class="big-tomato-clock">
      <div class="big-tomato-icon">
        <svg class="big-progress-ring" width="200" height="200">
          <circle class="bg" cx="100" cy="100" r="80" />
          <circle class="progress" cx="100" cy="100" r="80" stroke-dasharray="502.655"
            :stroke-dashoffset="bigProgressOffset" />
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
        <svg class="progress-ring" width="120" height="120">
          <circle class="bg" cx="60" cy="60" r="45" />
          <circle class="progress" cx="60" cy="60" r="45" stroke-dasharray="282.743"
            :stroke-dashoffset="progressOffset" />
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
        <input v-model="customMinutes" type="number" min="1" max="120" placeholder="分钟"
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
import { ref, computed, onUnmounted } from 'vue'

export default {
  name: 'TomatoTimer',
  setup() {
    const timeLeft = ref(0)
    const isRunning = ref(false)
    const isPaused = ref(false)
    const selectedTime = ref(0)
    const customMinutes = ref('')
    let timer = null

    const timeOptions = [
      { label: '15分钟', value: 15 * 60 },
      { label: '30分钟', value: 30 * 60 },
      { label: '1小时', value: 60 * 60 }
    ]

    const statusText = computed(() => {
      if (timeLeft.value === 0) return '选择时间开始专注'
      if (isRunning.value) return '专注中...'
      if (isPaused.value) return '已暂停'
      return '准备开始'
    })

    const progressOffset = computed(() => {
      if (selectedTime.value === 0) return 282.743
      // 圆周长 = 2 * π * r = 2 * 3.14159 * 45 ≈ 282.743
      const circumference = 2 * Math.PI * 45
      const progress = (timeLeft.value / selectedTime.value) * circumference
      return progress
    })

    const bigProgressOffset = computed(() => {
      if (selectedTime.value === 0) return 502.655
      // 圆周长 = 2 * π * r = 2 * 3.14159 * 80 ≈ 502.655
      const circumference = 2 * Math.PI * 80
      const progress = (timeLeft.value / selectedTime.value) * circumference
      return progress
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
      if (minutes && minutes > 0 && minutes <= 120) {
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

    // 请求通知权限
    if ('Notification' in window && Notification.permission === 'default') {
      Notification.requestPermission()
    }

    // 组件卸载时清理定时器
    onUnmounted(() => {
      if (timer) {
        clearInterval(timer)
      }
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
      formatTime,
      selectTime,
      startCustomTimer,
      startTimer,
      pauseTimer,
      resumeTimer,
      resetTimer
    }
  }
}
</script>