<script setup lang="ts">
import { onMounted, ref } from 'vue'

const props = defineProps<{
  steamId: string
}>()

const reviewImageRef = ref(null)

const isLoading = ref(true)
const copySuccess = ref(false)
const reviewText = ref('')
const isGeneratingImage = ref(false)

// 模拟从API获取数据
onMounted(() => {
  // 实际项目中应该调用真实的API
  setTimeout(() => {
    generateMockReview()
    isLoading.value = false
  }, 1500)
})

// 生成模拟的锐评内容
function generateMockReview() {
  reviewText.value = `作为一位Steam游戏玩家，你的游戏库反映了独特的品味和偏好。

你似乎偏爱策略和角色扮演类游戏，特别是那些提供深度沉浸体验的作品。《文明VI》和《全面战争》系列在你的游戏时间中占据显著比例，表明你享受宏观决策和历史模拟的复杂性。

值得注意的是，你对独立游戏的探索精神。《空洞骑士》和《星露谷物语》等作品在你的收藏中占有一席之地，显示你欣赏创新游戏设计和独特叙事的能力。

然而，你的游戏库中缺乏多样性的竞技类游戏，这可能意味着你更喜欢个人体验而非竞争环境。考虑尝试一些高质量的合作或轻度竞技游戏，可能会为你的游戏体验增添新的维度。`
}

// 复制锐评文本
function copyReview() {
  navigator.clipboard.writeText(reviewText.value)
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

    <!-- 结果展示 -->
    <div v-else class="rounded-xl bg-white shadow-lg overflow-hidden dark:bg-gray-800">
      <!-- 头部 -->
      <div class="bg-gradient-to-r px-4 py-3 from-blue-600 to-indigo-600 sm:px-6 sm:py-4">
        <h1 class="text-lg text-white font-bold flex gap-2 items-center sm:text-xl">
          <span class="i-carbon-analytics" />
          Steam游戏库AI锐评
        </h1>
        <p class="text-xs text-blue-100 mt-1 sm:text-sm">
          Steam ID: {{ steamId }}
        </p>
      </div>

      <!-- 锐评内容 -->
      <div class="p-4 sm:p-6">
        <div class="mb-4 p-3 rounded-lg bg-gray-50 sm:mb-6 sm:p-5 dark:bg-gray-900">
          <p class="text-sm text-gray-800 leading-relaxed whitespace-pre-line sm:text-base dark:text-gray-200">
            {{ reviewText }}
          </p>
        </div>

        <!-- 操作按钮 -->
        <div class="flex flex-col gap-3 items-center justify-between sm:flex-row">
          <button
            class="text-sm px-3 py-1.5 rounded-lg bg-gray-100 flex gap-2 w-full transition-colors items-center justify-center sm:px-4 sm:py-2 dark:bg-gray-700 hover:bg-gray-200 sm:w-auto sm:justify-start dark:hover:bg-gray-600"
            @click="goBack"
          >
            <span class="i-carbon-arrow-left" />
            重新生成
          </button>

          <div class="flex gap-2 w-full sm:gap-3 sm:w-auto">
            <button
              class="text-sm text-white px-3 py-1.5 rounded-lg bg-blue-600 flex flex-1 gap-2 transition-colors items-center justify-center sm:px-4 sm:py-2 hover:bg-blue-700 sm:flex-auto"
              @click="copyReview"
            >
              <span v-if="copySuccess" class="i-carbon-checkmark" />
              <span v-else class="i-carbon-copy" />
              {{ copySuccess ? '已复制' : '复制文本' }}
            </button>

            <!-- 生成图片按钮 -->
            <button
              class="text-sm text-white px-3 py-1.5 rounded-lg bg-green-600 flex flex-1 gap-2 transition-colors items-center justify-center sm:px-4 sm:py-2 hover:bg-green-700 sm:flex-auto"
              :disabled="isGeneratingImage"
              :class="{ 'opacity-70 cursor-not-allowed': isGeneratingImage }"
              @click="reviewImageRef?.generateImage()"
            >
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
  <ReviewImageGenerator
    ref="reviewImageRef"
    v-model:is-generating-image="isGeneratingImage"
    :review-text="reviewText"
    :steam-id="steamId"
  />
</template>
