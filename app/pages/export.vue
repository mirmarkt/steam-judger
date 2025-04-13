<script setup lang="ts">
import { ref } from 'vue'
import { promptV3 } from '~/prompts/v3'

const steamId = ref('')
const isLoading = ref(false)
const error = ref('')
const exportData = ref<any | null>(null)
const selectedPrompt = ref<number | null>(null)
const promptList = ref([
  { id: 1, name: '默认Prompt', content: promptV3() },
])
const copySuccess = ref(false)

// 提取SteamID函数，与SteamIdInput组件保持一致
function extractSteamIdFromUrl(url: string) {
  const _url = url.trim()
  const steamIdMatch = _url.match(/steam_id=(\d+)|profiles\/(\d+)/)
  if (steamIdMatch) {
    return steamIdMatch[1]! || steamIdMatch[2]!
  }
  return url
}

// 处理表单提交
async function handleSubmit() {
  if (!steamId.value)
    return

  error.value = ''
  isLoading.value = true
  exportData.value = null

  const _steamId = extractSteamIdFromUrl(steamId.value)

  try {
    // 调用获取Steam游戏数据API
    const response = await fetch(`/api/query/${encodeURIComponent(_steamId)}`)

    if (!response.ok) {
      const errorData = await response.text()
      throw new Error(errorData || '获取游戏数据失败')
    }

    const data = await response.json()

    // 检查游戏数量，如果为0则提示用户
    if (data.gameCount === 0) {
      error.value = '您的Steam账号游戏数据为空，可能是Steam接口正在被限制访问，请稍后再试'
      isLoading.value = false
      return
    }

    // 设置导出数据
    exportData.value = data
  }
  catch (err: any) {
    error.value = err.message || '获取数据失败，请稍后再试'
    console.error('获取数据失败', err)
  }
  finally {
    isLoading.value = false
  }
}

// 下载数据到本地
function downloadData() {
  if (!exportData.value)
    return

  const dataStr = JSON.stringify(exportData.value, null, 2)
  const dataBlob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(dataBlob)

  const a = document.createElement('a')
  a.href = url
  a.download = `steam-data-${steamId.value}.json`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}

// 复制Prompt到剪贴板
function copyPrompt() {
  if (!selectedPrompt.value)
    return

  const promptContent = promptList.value.find(p => p.id === selectedPrompt.value)?.content || ''
  navigator.clipboard.writeText(promptContent)
    .then(() => {
      copySuccess.value = true
      setTimeout(() => {
        copySuccess.value = false
      }, 2000)
    })
    .catch((err) => {
      console.error('复制失败', err)
    })
}
</script>

<template>
  <div class="py-6 h-full sm:py-10">
    <div class="mx-auto px-4 sm:px-6 container">
      <div class="mb-6 text-center sm:mb-10">
        <h1 class="text-2xl font-bold mb-2 sm:text-3xl">
          <span class="text-blue-500 mr-1">Steam</span>
          <span>数据</span>
          <span class="text-black ml-1 px-2 py-1 rounded bg-yellow inline-block">导出</span>
        </h1>
        <p class="text-sm text-gray-500 mx-auto max-w-md sm:text-base dark:text-gray-400">
          导出Steam游戏数据并获取自定义Prompt
        </p>
      </div>

      <!-- Steam ID输入表单 -->
      <div class="mx-auto mb-6 p-4 rounded-xl bg-white max-w-md w-full shadow-lg sm:mb-8 sm:p-6 dark:bg-gray-800">
        <form class="space-y-4" @submit.prevent="handleSubmit">
          <div>
            <label for="steamId" class="text-sm text-gray-700 font-medium mb-1 block dark:text-gray-300">Steam ID</label>
            <input
              id="steamId"
              v-model="steamId"
              type="text"
              placeholder="输入64位Steam ID或Steam个人资料URL"
              class="focus:outline-none px-3 py-2 border border-gray-300 rounded-md w-full shadow-sm dark:text-white dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 focus:ring-2 focus:ring-blue-500"
              :disabled="isLoading"
            >
          </div>

          <div>
            <button
              type="submit"
              class="focus:outline-none text-sm text-white font-medium px-4 py-2 border border-transparent rounded-md bg-blue-600 flex w-full shadow-sm justify-center hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 dark:focus:ring-offset-gray-800"
              :disabled="isLoading || !steamId"
            >
              <span v-if="isLoading" class="i-carbon-circle-dash mr-2 animate-spin" />
              {{ isLoading ? '获取中...' : '获取数据' }}
            </button>
          </div>

          <div v-if="error" class="text-sm text-red-500 mt-2">
            {{ error }}
          </div>
        </form>
      </div>

      <!-- 导出数据结果 -->
      <div v-if="exportData" class="mx-auto mb-6 p-4 rounded-xl bg-white max-w-2xl w-full shadow-lg sm:mb-8 sm:p-6 dark:bg-gray-800">
        <div class="mb-4 flex items-center justify-between">
          <h2 class="text-lg font-medium">
            数据导出
          </h2>
          <button
            class="focus:outline-none text-sm text-white font-medium px-3 py-1.5 rounded-md bg-green-600 flex gap-1 items-center hover:bg-green-700 focus:ring-2 focus:ring-green-500 focus:ring-offset-2 dark:focus:ring-offset-gray-800"
            @click="downloadData"
          >
            <span class="i-carbon-download" />
            下载JSON
          </button>
        </div>

        <div class="text-sm font-mono p-3 rounded-md bg-gray-100 max-h-60 overflow-y-auto dark:bg-gray-700">
          <pre>{{ JSON.stringify(exportData, null, 2) }}</pre>
        </div>

        <div class="text-sm text-gray-500 mt-4 dark:text-gray-400">
          <p>游戏总数: {{ exportData.gameCount || 0 }}</p>
          <p>Steam ID: {{ exportData.steamId }}</p>
        </div>
      </div>

      <!-- 自定义Prompt选择 -->
      <div class="mx-auto mb-6 p-4 rounded-xl bg-white max-w-2xl w-full shadow-lg sm:mb-8 sm:p-6 dark:bg-gray-800">
        <h2 class="text-lg font-medium mb-4">
          自定义Prompt
        </h2>

        <div class="space-y-4">
          <div>
            <label for="promptSelect" class="text-sm text-gray-700 font-medium mb-1 block dark:text-gray-300">选择Prompt模板</label>
            <select
              id="promptSelect"
              v-model="selectedPrompt"
              class="focus:outline-none px-3 py-2 border border-gray-300 rounded-md w-full shadow-sm dark:text-white dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 focus:ring-2 focus:ring-blue-500"
            >
              <option :value="null">
                -- 请选择 --
              </option>
              <option v-for="prompt in promptList" :key="prompt.id" :value="prompt.id">
                {{ prompt.name }}
              </option>
            </select>
          </div>

          <div v-if="selectedPrompt">
            <div class="mb-2 flex items-center justify-between">
              <label class="text-sm text-gray-700 font-medium block dark:text-gray-300">Prompt内容</label>
              <button
                class="focus:outline-none text-xs text-white font-medium px-2 py-1 rounded-md bg-blue-600 flex gap-1 items-center hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 dark:focus:ring-offset-gray-800"
                @click="copyPrompt"
              >
                <span v-if="!copySuccess" class="i-carbon-copy" />
                <span v-else class="i-carbon-checkmark text-green-400" />
                {{ copySuccess ? '已复制' : '复制' }}
              </button>
            </div>

            <div class="text-sm p-3 rounded-md bg-gray-100 dark:bg-gray-700">
              {{ promptList.find(p => p.id === selectedPrompt)?.content }}
            </div>
          </div>
        </div>
      </div>

      <!-- 返回首页链接 -->
      <div class="text-center">
        <NuxtLink to="/" class="text-blue-500 inline-flex gap-1 items-center hover:underline">
          <span class="i-carbon-arrow-left" />
          返回首页
        </NuxtLink>
      </div>
    </div>
  </div>
</template>
