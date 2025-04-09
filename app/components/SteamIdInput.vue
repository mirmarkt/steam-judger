<script setup lang="ts">
const steamId = ref('')
const isLoading = ref(false)
const router = useRouter()

function handleSubmit() {
  if (!steamId.value)
    return

  isLoading.value = true

  // 模拟API调用，实际项目中应该调用真实的API
  setTimeout(() => {
    isLoading.value = false
    // 跳转到结果页面，并传递steamId参数
    router.push(`/review?steamId=${encodeURIComponent(steamId.value)}`)
  }, 1000)
}
</script>

<template>
  <div class="mx-auto px-4 max-w-md w-full sm:px-0">
    <div class="mb-4 text-left sm:mb-6">
      <label for="steam-id" class="text-sm text-gray-700 font-medium mb-2 block dark:text-gray-200">
        <span class="flex gap-1 items-center">
          <span class="i-carbon-game-console text-lg" />
          Steam用户名/ID
        </span>
      </label>
      <div class="relative">
        <input
          id="steam-id"
          v-model="steamId"
          type="text"
          placeholder="请输入您的Steam用户名或ID"
          class="outline-none text-sm px-3 py-2.5 border border-gray-200 rounded-lg bg-white w-full transition sm:text-base sm:px-4 sm:py-3 dark:border-gray-700 focus:border-transparent dark:bg-gray-800 focus:ring-2 focus:ring-blue-500"
          :disabled="isLoading"
          @keyup.enter="handleSubmit"
        >
      </div>
      <div class="text-xs text-gray-500 mt-2 flex gap-1 items-center dark:text-gray-400">
        <span class="i-carbon-information" />
        <a href="https://help.steampowered.com/zh-cn/faqs/view/2816-BE67-5B69-0FEC" target="_blank" class="text-blue-500 hover:underline">如何查找我的Steam ID?</a>
      </div>
    </div>

    <button
      class="text-sm text-white font-medium px-4 py-2.5 rounded-lg bg-blue-600 flex gap-2 w-full transition-colors items-center justify-center sm:text-base sm:px-6 sm:py-3 hover:bg-blue-700"
      :class="{ 'opacity-70 cursor-not-allowed': isLoading }"
      :disabled="isLoading || !steamId"
      @click="handleSubmit"
    >
      <span v-if="isLoading" class="i-carbon-circle-dash animate-spin" />
      <span v-else class="i-carbon-analytics" />
      {{ isLoading ? '生成中...' : '生成AI锐评' }}
    </button>
  </div>
</template>
