<script setup lang="ts">
import MarkdownIt from 'markdown-it'
import { computed, onMounted, ref } from 'vue'

const props = defineProps<{
  steamId: string
  dataId: string
}>()

const reviewImageRef = ref<any>(null)

const isLoading = ref(true)
const copySuccess = ref(false)
const reviewText = ref('')
const isGeneratingImage = ref(false)
const error = ref('')
const isStreamingComplete = ref(false)
const modelInfo = ref<{ modelName: string, version: string } | null>(null)
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

// 初始化markdown-it解析器
const md = new MarkdownIt()

// 解析Markdown内容为HTML
const parsedReviewHtml = computed(() => {
  return reviewText.value ? md.render(reviewText.value) : ''
})

// 从API获取数据
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

// 获取模型信息
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

// 获取Steam用户信息
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

// 获取AI分析结果（流式传输）
async function fetchAnalysis() {
  isLoading.value = true
  error.value = ''
  reviewText.value = ''
  isStreamingComplete.value = false

  try {
    const response = await fetch(`/api/analyze/${encodeURIComponent(props.dataId)}`)

    if (!response.ok) {
      const errorData = await response.text()
      throw new Error(errorData || '获取分析结果失败')
    }

    if (!response.body) {
      throw new Error('浏览器不支持流式传输')
    }

    // 处理流式响应
    const reader = response.body.getReader()
    const decoder = new TextDecoder()

    isLoading.value = false

    // 读取流数据
    while (true) {
      const { done, value } = await reader.read()

      if (done) {
        isStreamingComplete.value = true
        break
      }

      // 解码并添加到文本中
      const text = decoder.decode(value, { stream: true })
      // 使用nextTick确保DOM更新
      reviewText.value += text
      // 强制Vue更新视图
      await nextTick()
    }
  }
  catch (err: any) {
    error.value = err.message || '获取分析结果失败，请稍后重试'
    isLoading.value = false
  }
}

// 复制锐评文本
function copyReview() {
  // 创建一个临时元素来获取纯文本内容（去除HTML标签）
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

// 返回首页
const router = useRouter()
function goBack() {
  router.push('/')
}
</script>

<template>
  <div class="mx-auto max-w-3xl w-full">
    <!-- 加载状态 -->
    <div v-if="isLoading" class="py-20 flex flex-col items-center justify-center">
      <div class="i-carbon-game-console text-6xl text-blue-500 mb-4 animate-pulse" />
      <p class="text-lg font-medium">
        正在分析Steam游戏库，生成AI锐评中...
      </p>
    </div>

    <!-- 错误状态 -->
    <div v-else-if="error" class="py-20 flex flex-col items-center justify-center">
      <div class="i-carbon-warning text-5xl text-red-500 mb-4" />
      <p class="text-lg text-red-500 font-medium mb-2">
        出错了
      </p>
      <p class="text-sm text-gray-600 mb-4 text-center max-w-md dark:text-gray-300">
        {{ error }}
      </p>
      <button
        class="text-sm text-white px-4 py-2 rounded-lg bg-blue-600 flex gap-2 transition-colors items-center justify-center hover:bg-blue-700"
        @click="goBack">
        <span class="i-carbon-arrow-left" />
        返回首页
      </button>
    </div>

    <!-- 结果展示 -->
    <div v-else class="rounded-xl bg-white shadow-lg overflow-hidden dark:bg-gray-800">
      <!-- 头部 -->
      <div class="bg-gradient-to-r px-4 py-3 from-blue-600 to-indigo-600 sm:px-6 sm:py-4">
        <div class="flex flex-col gap-3 items-center md:flex-row md:items-start md:justify-between">
          <!-- 用户信息区域 -->
          <div class="flex gap-3 items-center">
            <!-- 用户头像 -->
            <div
              class="rounded-full bg-blue-400/30 flex h-12 w-12 items-center justify-center overflow-hidden sm:h-14 sm:w-14">
              <img v-if="userInfo?.avatarMediumUrl" :src="userInfo.avatarMediumUrl" alt="Steam头像"
                class="h-full w-full object-cover">
              <span v-else class="i-carbon-user text-xl text-white" />
            </div>

            <!-- 用户名称和ID -->
            <div>
              <h2 class="text-lg font-bold text-center sm:text-xl dark:text-white md:text-left">
                {{ userInfo?.personaName || '游戏玩家' }}
              </h2>
              <p
                class="text-xs text-gray flex gap-1 items-center justify-center sm:text-sm dark:text-blue-100/70 md:justify-start">
                <span class="i-carbon-logo-steam" />
                {{ steamId }}
              </p>
            </div>
          </div>

          <!-- 报告信息 -->
          <div class="text-center md:text-right">
            <h1 class="text-lg text-white font-bold flex gap-2 items-center justify-center sm:text-xl md:justify-end">
              <span class="i-carbon-analytics" />
              AI锐评报告
              <span v-if="modelInfo" class="text-sm px-2 py-1 rounded-lg bg-blue-500/80">{{ modelInfo.version }}</span>
            </h1>
            <p v-if="modelInfo"
              class="text-sm text-gray mt-1 flex gap-1 items-center justify-center sm:text-sm dark:text-blue-100/70 md:justify-end">
              <span class="i-carbon-ai-generate text-xs" />
              {{ modelInfo.modelName }}
            </p>
          </div>
        </div>
      </div>

      <!-- 锐评内容 -->
      <div class="p-4 sm:p-6">
        <div class="mb-4 p-3 rounded-lg bg-gray-50 sm:mb-6 sm:p-5 dark:bg-gray-900">
          <!-- 流式加载中 -->
          <div v-if="reviewText && !isStreamingComplete" class="mb-2 flex items-center justify-end">
            <span class="text-xs text-blue-500 flex gap-1 items-center">
              <span class="i-carbon-circle-dash animate-spin" />
              正在生成中...
            </span>
          </div>
          <div v-if="reviewText"
            class="markdown-content text-sm text-gray-800 leading-relaxed sm:text-base dark:text-gray-200"
            v-html="parsedReviewHtml" />
          <p v-else class="text-sm text-gray-800 leading-relaxed sm:text-base dark:text-gray-200">
            正在生成AI锐评...
          </p>
        </div>

        <!-- 操作按钮 -->
        <div class="flex flex-col gap-3 items-center justify-between sm:flex-row">
          <button
            class="text-sm px-3 py-1.5 rounded-lg bg-gray-100 flex gap-2 w-full transition-colors items-center justify-center sm:px-4 sm:py-2 dark:bg-gray-700 hover:bg-gray-200 sm:w-auto sm:justify-start dark:hover:bg-gray-600"
            @click="goBack">
            <span class="i-carbon-arrow-left" />
            重新生成
          </button>

          <div class="flex gap-2 w-full sm:gap-3 sm:w-auto">
            <button
              class="text-sm text-white px-3 py-1.5 rounded-lg bg-blue-600 flex flex-1 gap-2 transition-colors items-center justify-center sm:px-4 sm:py-2 hover:bg-blue-700 sm:flex-auto"
              @click="copyReview">
              <span v-if="copySuccess" class="i-carbon-checkmark" />
              <span v-else class="i-carbon-copy" />
              {{ copySuccess ? '已复制' : '复制文本' }}
            </button>

            <!-- 生成图片按钮 -->
            <button
              class="text-sm text-white px-3 py-1.5 rounded-lg bg-green-600 flex flex-1 gap-2 transition-colors items-center justify-center sm:px-4 sm:py-2 hover:bg-green-700 sm:flex-auto"
              :disabled="isGeneratingImage" :class="{ 'opacity-70 cursor-not-allowed': isGeneratingImage }"
              @click="reviewImageRef?.generateImage()">
              <span v-if="isGeneratingImage" class="i-carbon-circle-dash animate-spin" />
              <span v-else class="i-carbon-share" />
              {{ isGeneratingImage ? '生成中...' : '生成图片' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 图片生成器组件 -->
  <ReviewImageGenerator ref="reviewImageRef" v-model:is-generating-image="isGeneratingImage" :review-text="reviewText"
    :steam-id="steamId" :model-info="modelInfo" :user-info="userInfo" />
</template>
