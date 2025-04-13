<script setup lang="ts">
import MarkdownIt from 'markdown-it'
import { computed, onMounted, onUnmounted, ref } from 'vue'
import { useRouter } from 'vue-router' // Show warning after 90 seconds

const props = defineProps<{
  steamId: string
  dataId: string
}>() // Assuming you use vue-router

// --- Constants ---
const STREAM_TIMEOUT_MS = 2 * 60 * 1000 // 2 minutes
const WARNING_THRESHOLD_MS = 90 * 1000

const reviewImageRef = ref<any>(null) // Assuming ReviewImageGenerator component exists

const isLoading = ref(true)
const copySuccess = ref(false)
const reviewText = ref('')
const isGeneratingImage = ref(false)
const error = ref('')
const isStreamingComplete = ref(false)
const showTimeoutWarning = ref(false) // New state for warning message
const modelInfo = ref<{ modelName: string, version: string } | null>(null)
const progress = ref(0)
const progressTimer = ref<ReturnType<typeof setInterval> | null>(null)
const timeoutTimerId = ref<ReturnType<typeof setTimeout> | null>(null) // Timer for the 2-min limit
const warningTimerId = ref<ReturnType<typeof setTimeout> | null>(null) // Timer for the warning message
// const retryCount = ref(0) // Keep if needed, but not used for retry logic here
const userInfo = ref<{
  steamId: string
  personaName: string
  profileUrl: string
  avatarIconUrl: string
  avatarMediumUrl: string
  avatarFullUrl: string
  personaState: number
  visibilityState: number
} | null>(null)
const isLoadingUserInfo = ref(false)

// Initialize markdown-it parser
const md = new MarkdownIt()

// Parse Markdown content to HTML
const parsedReviewHtml = computed(() => {
  return reviewText.value ? md.render(reviewText.value) : ''
})

// --- Lifecycle Hooks ---
onMounted(() => {
  if (props.dataId) {
    fetchAnalysis()
    fetchModelInfo()
    fetchUserInfo()
  }
  else {
    error.value = '缺少必要的数据ID参数'
    isLoading.value = false
  }
})

onUnmounted(() => {
  // Clean up timers when component is destroyed
  clearAllTimers()
})

// --- Methods ---

// Clear all timers
function clearAllTimers() {
  if (progressTimer.value) {
    clearInterval(progressTimer.value)
    progressTimer.value = null
  }
  if (timeoutTimerId.value) {
    clearTimeout(timeoutTimerId.value)
    timeoutTimerId.value = null
  }
  if (warningTimerId.value) {
    clearTimeout(warningTimerId.value)
    warningTimerId.value = null
  }
}

// Fetch model info
async function fetchModelInfo() {
  try {
    const response = await fetch('/api/model')
    if (response.ok) {
      modelInfo.value = await response.json()
    }
  }
  catch (err) {
    console.error('获取模型信息失败', err)
  }
}

// Fetch Steam user info
async function fetchUserInfo() {
  if (!props.steamId)
    return

  isLoadingUserInfo.value = true
  try {
    const response = await fetch(`/api/user/${encodeURIComponent(props.steamId)}`)
    if (response.ok) {
      userInfo.value = await response.json()
    }
  }
  catch (err) {
    console.error('获取Steam用户信息失败', err)
  }
  finally {
    isLoadingUserInfo.value = false
  }
}

// Fetch AI analysis result (streaming)
async function fetchAnalysis() {
  isLoading.value = true
  error.value = ''
  reviewText.value = ''
  isStreamingComplete.value = false
  showTimeoutWarning.value = false // Reset warning state
  progress.value = 0
  // retryCount.value++ // Increment if needed

  // Clear previous timers before starting new ones
  clearAllTimers()

  try {
    const response = await fetch(`/api/analyze/${encodeURIComponent(props.dataId)}`)

    if (!response.ok) {
      const errorData = await response.text()
      throw new Error(errorData || '获取分析结果失败')
    }

    if (!response.body) {
      throw new Error('浏览器不支持流式传输')
    }

    // Handle streaming response
    const reader = response.body.getReader()
    const decoder = new TextDecoder()

    // Start progress simulation (slightly adjusted)
    progressTimer.value = setInterval(() => {
      if (progress.value < 95 && !isStreamingComplete.value) { // Stop near the end
        // Slower progress increase for a more realistic feel
        progress.value += Math.max(0.5, Math.random() * 1.5)
        progress.value = Math.min(progress.value, 95) // Cap at 95 before completion
      }
      else if (isStreamingComplete.value) {
        // Ensure it hits 100 only on actual completion
        progress.value = 100
        if (progressTimer.value)
          clearInterval(progressTimer.value)
      }
    }, 800) // Update interval

    // --- Timeout and Warning Logic ---
    // Start warning timer
    warningTimerId.value = setTimeout(() => {
      if (!isStreamingComplete.value) {
        showTimeoutWarning.value = true
      }
    }, WARNING_THRESHOLD_MS)

    // Start main timeout timer
    timeoutTimerId.value = setTimeout(() => {
      if (!isStreamingComplete.value) {
        console.error(`Streaming timed out after ${STREAM_TIMEOUT_MS / 1000} seconds.`)
        error.value = `内容生成时间过长（超过${STREAM_TIMEOUT_MS / 1000 / 60}分钟），已自动中断。请返回首页重试。`
        // Force stop processing and cleanup
        isStreamingComplete.value = true // Mark as 'complete' to stop UI updates
        clearAllTimers() // Clear all timers
        progress.value = 0 // Reset or hide progress bar on error
        // Note: This doesn't truly abort the fetch, just stops processing the stream.
        // For full abort, AbortController would be needed, adding complexity.
      }
    }, STREAM_TIMEOUT_MS)
    // --- End Timeout Logic ---

    isLoading.value = false // Show content area now

    // Read stream data
    while (true) {
      // Check if timeout error occurred before reading next chunk
      if (error.value) {
        break // Exit loop if timeout happened
      }

      const { done, value } = await reader.read()

      if (done) {
        if (!error.value) { // Only mark as complete if no timeout error occurred
          isStreamingComplete.value = true
          progress.value = 100 // Ensure progress hits 100%
          showTimeoutWarning.value = false // Hide warning on successful completion
        }
        clearAllTimers() // Clear timers on successful completion
        break
      }

      // Decode and append text
      const text = decoder.decode(value, { stream: true })
      reviewText.value += text
      // Ensure DOM updates if needed (usually not necessary with ref changes)
      // await nextTick()
    }
  }
  catch (err: any) {
    if (!error.value) { // Avoid overwriting timeout error
      error.value = err.message || '获取分析结果失败，请稍后重试'
    }
    isLoading.value = false
    clearAllTimers() // Clean up timers on fetch error
    progress.value = 0 // Reset progress on error
  }
  finally {
    // Final cleanup check for timers, although should be handled above
    if (!isStreamingComplete.value && !error.value) {
      // Handle cases where loop might exit unexpectedly without error/completion?
      // This block might not be strictly necessary if logic above is sound.
    }
    // Ensure loading is false if it hasn't been set already
    if (isLoading.value)
        isLoading.value = false
  }
}

// Copy review text
function copyReview() {
  const tempElement = document.createElement('div')
  tempElement.innerHTML = parsedReviewHtml.value
  // eslint-disable-next-line unicorn/prefer-dom-node-text-content
  const textContent = tempElement.textContent || tempElement.innerText || reviewText.value

  navigator.clipboard.writeText(textContent)
  copySuccess.value = true
  setTimeout(() => {
    copySuccess.value = false
  }, 2000)
}

// Go back (e.g., to home or previous step)
const router = useRouter()
function goBack() {
  router.push('/') // Or router.back() if appropriate
}
</script>

<template>
  <div class="mx-auto max-w-3xl w-full">
    <!-- Loading State -->
    <div v-if="isLoading" class="px-4 py-20 text-center flex flex-col items-center justify-center">
      <div class="i-carbon-game-console text-6xl text-blue-500 mb-4 animate-pulse dark:text-blue-400" />
      <p class="text-lg text-gray-700 font-medium dark:text-gray-300">
        正在分析Steam游戏库，生成AI锐评中...
      </p>
      <p class="text-sm text-gray-500 mt-2 dark:text-gray-400">
        这可能需要1-2分钟，请稍候。
      </p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="px-4 py-20 text-center flex flex-col items-center justify-center">
      <div class="i-carbon-warning-alt text-5xl text-red-500 mb-4 dark:text-red-400" />
      <p class="text-lg text-red-600 font-medium mb-2 dark:text-red-400">
        出错了
      </p>
      <p class="text-sm text-gray-600 mb-6 text-center max-w-md dark:text-gray-300">
        {{ error }} <!-- Display specific error message (including timeout) -->
      </p>
      <button
        class="text-sm text-white px-4 py-2 rounded-lg bg-blue-600 flex gap-2 transition-colors items-center justify-center dark:bg-blue-500 hover:bg-blue-700 dark:hover:bg-blue-600"
        @click="goBack"
      >
        <span class="i-carbon-arrow-left" />
        返回首页重试
      </button>
    </div>

    <!-- Result Display -->
    <div v-else class="rounded-xl bg-white shadow-lg overflow-hidden dark:bg-gray-800">
      <!-- Progress Bar and Header Section -->
      <div class="border-b border-gray-200 dark:border-gray-700">
        <!-- Progress Bar (only show while streaming and no error) -->
        <div v-if="!isStreamingComplete && !error" class="px-4 pt-4 sm:px-6">
          <div class="rounded-full bg-gray-200 h-2 w-full overflow-hidden dark:bg-gray-600">
            <div
              class="bg-blue-500 h-full transition-all duration-500 ease-linear"
              :style="{ width: `${progress}%` }"
            />
          </div>
          <!-- Timeout Warning Message -->
          <div v-if="showTimeoutWarning && !isStreamingComplete && !error" class="text-sm text-yellow-800 mt-3 p-3 border border-yellow-300 rounded-md bg-yellow-100 flex gap-2 items-start dark:text-yellow-300 dark:border-yellow-700/50 dark:bg-yellow-900/30">
            <span class="i-carbon-warning-alt text-base mt-0.5 flex-shrink-0" />
            <span>生成时间可能较长。如果超过 {{ STREAM_TIMEOUT_MS / 1000 / 60 }} 分钟，处理将自动中断，届时请尝试重新生成。</span>
          </div>
        </div>

        <!-- Header -->
        <div class="px-4 py-4 sm:px-6 sm:py-5">
          <!-- Header content (User Info, Report Title) -->
          <div class="flex flex-col gap-3 items-center md:flex-row md:items-start md:justify-between">
            <!-- User Info Area -->
            <div class="flex gap-3 items-center">
              <!-- Avatar -->
              <div class="rounded-full bg-gray-200 flex flex-shrink-0 h-12 w-12 items-center justify-center overflow-hidden dark:bg-gray-700 sm:h-14 sm:w-14">
                <img v-if="userInfo?.avatarMediumUrl" :src="userInfo.avatarMediumUrl" alt="Steam Avatar" class="h-full w-full object-cover">
                <span v-else class="i-carbon-user text-2xl text-gray-500 dark:text-gray-400" />
              </div>
              <!-- Name & ID -->
              <div>
                <h2 class="text-lg text-gray-900 font-semibold text-center sm:text-xl dark:text-white md:text-left">
                  {{ userInfo?.personaName || '游戏玩家' }}
                </h2>
                <p class="text-xs text-gray-500 flex gap-1 items-center justify-center sm:text-sm dark:text-gray-400 md:justify-start">
                  <span class="i-carbon-logo-steam" />
                  {{ steamId }}
                </p>
              </div>
            </div>
            <!-- Report Info -->
            <div class="mt-2 text-center md:mt-0 md:text-right">
              <h1 class="text-lg font-semibold flex gap-2 items-center justify-center sm:text-xl dark:text-white md:justify-end">
                <span class="underline decoration-blue-500 decoration-wavy dark:decoration-blue-400">AI锐评报告</span>
                <span v-if="modelInfo" class="text-xs text-blue-800 px-1.5 py-0.5 rounded bg-blue-100 dark:text-blue-300 dark:bg-blue-900/50">{{ modelInfo.version }}</span>
              </h1>
              <p v-if="modelInfo" class="text-xs text-gray-500 mt-1 flex gap-1 items-center justify-center sm:text-sm dark:text-gray-400 md:justify-end">
                <span class="i-carbon-chip text-xs" /> <!-- Changed icon -->
                {{ modelInfo.modelName }}
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- Review Content -->
      <div class="p-4 sm:p-6">
        <div class="mb-4 p-4 rounded-lg bg-gray-50 ring-1 ring-gray-100 ring-inset sm:mb-6 sm:p-5 dark:bg-gray-900/50 dark:ring-gray-700/50">
          <!-- Streaming Indicator -->
          <div v-if="reviewText && !isStreamingComplete && !error" class="mb-2 flex items-center justify-end">
            <span class="text-xs text-blue-600 font-medium flex gap-1 items-center dark:text-blue-400">
              <span class="i-carbon-circle-dash animate-spin" />
              正在生成中...
            </span>
          </div>
          <!-- Content Area -->
          <div
            v-if="reviewText"
            class="markdown-content prose-sm sm:prose-base text-gray-800 max-w-none prose dark:text-gray-200 dark:prose-invert"
            v-html="parsedReviewHtml"
          />
          <!-- Placeholder while waiting for first chunk -->
          <p v-else-if="!error" class="text-sm text-gray-500 italic dark:text-gray-400">
            AI正在思考，请耐心等待第一段内容...
          </p>
          <!-- Show nothing here if there was an error (error shown above) -->
        </div>

        <!-- Action Buttons -->
        <div class="flex flex-col gap-3 items-center justify-between sm:flex-row">
          <button
            class="text-sm text-gray-700 px-4 py-2 rounded-lg bg-gray-100 flex gap-2 w-full transition-colors items-center justify-center dark:text-gray-200 dark:bg-gray-700 hover:bg-gray-200 sm:w-auto dark:hover:bg-gray-600"
            @click="goBack"
          >
            <span class="i-carbon-arrow-left" />
            <!-- Changed text slightly -->
            返回首页
          </button>

          <div class="flex gap-2 w-full sm:gap-3 sm:w-auto">
            <button
              class="text-sm text-white px-4 py-2 rounded-lg bg-blue-600 flex flex-1 gap-2 transition-colors items-center justify-center dark:bg-blue-500 hover:bg-blue-700 sm:flex-auto dark:hover:bg-blue-600"
              :disabled="!reviewText || !!error"
              :class="{ 'opacity-50 cursor-not-allowed': !reviewText || error }"
              @click="copyReview"
            >
              <span v-if="copySuccess" class="i-carbon-checkmark" />
              <span v-else class="i-carbon-copy" />
              {{ copySuccess ? '已复制' : '复制文本' }}
            </button>

            <!-- Generate Image Button -->
            <button
              class="text-sm text-white px-4 py-2 rounded-lg bg-green-600 flex flex-1 gap-2 transition-colors items-center justify-center dark:bg-green-500 hover:bg-green-700 sm:flex-auto dark:hover:bg-green-600"
              :disabled="isGeneratingImage || !reviewText || !!error"
              :class="{ 'opacity-50 cursor-not-allowed': isGeneratingImage || !reviewText || error }"
              @click="reviewImageRef?.generateImage()"
            >
              <span v-if="isGeneratingImage" class="i-carbon-circle-dash animate-spin" />
              <span v-else class="i-carbon-image" /> <!-- Changed icon -->
              {{ isGeneratingImage ? '生成中...' : '生成图片' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <ReviewImageGenerator
    ref="reviewImageRef" v-model:is-generating-image="isGeneratingImage" :review-text="reviewText"
    :steam-id="steamId" :model-info="modelInfo" :user-info="userInfo"
  />
</template>
