<script setup lang="ts">
import { appDescription } from '~/constants'
</script>

<template>
  <div class="px-4 flex flex-col h-full items-center justify-center sm:px-6">
    <!-- 添加Steam接口限制提示 -->
    <div class="text-amber-800 mb-4 p-3 border border-amber-200 rounded-lg bg-amber-50 flex gap-2 max-w-md w-full shadow-sm items-center dark:text-amber-200 sm:mb-6 dark:border-amber-700/50 dark:bg-amber-900/30">
      <span class="i-carbon-warning-alt text-lg text-amber-500 flex-shrink-0 dark:text-amber-400" />
      <p class="text-sm font-medium">
        当前Steam接口高频访问已被官方限制，请耐心等待
      </p>
    </div>
    <div class="mb-6 text-center sm:mb-10">
      <div class="mb-3 flex justify-center sm:mb-4">
        <div class="i-carbon-game-console text-5xl text-blue-500 sm:text-6xl" />
      </div>
      <h1 class="text-2xl font-bold mb-2 sm:text-3xl">
        <!-- {{ appName }} -->
        <span class="text-blue-500 mr-1">Steam</span>
        <span>游戏</span>
        <span class="text-black ml-1 px-2 py-1 rounded bg-yellow inline-block">锐评</span>
      </h1>
      <p class="text-sm text-gray-500 mx-auto max-w-md sm:text-base dark:text-gray-400">
        {{ appDescription }}
      </p>
    </div>

    <div class="mb-6 p-4 rounded-xl bg-white max-w-md w-full shadow-lg sm:mb-8 sm:p-6 dark:bg-gray-800">
      <SteamIdInput />
    </div>

    <div class="text-xs text-gray-500 px-4 py-2.5 rounded-full bg-gray-50/80 flex gap-2 shadow-sm transition-all duration-300 items-center backdrop-blur-md sm:text-sm dark:text-gray-400 dark:bg-gray-800/60 hover:bg-gray-50/90 hover:shadow dark:hover:bg-gray-800/70">
      <span class="i-carbon-location-heart-filled text-blue-500" />
      <span>如果您觉得吐槽姬给您带来了快乐，欢迎
        <NuxtLink to="/sponsor" class="text-blue-500 font-medium transition-colors hover:text-blue-600 hover:underline">支持我们</NuxtLink>
        继续开发
      </span>
    </div>
  </div>
</template>
