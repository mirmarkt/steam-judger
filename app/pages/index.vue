<script setup lang="ts">
import { appDescription, appName } from '~/constants'
</script>

<template>
  <div class="px-4 flex flex-col h-full items-center justify-center sm:px-6">
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

    <div class="text-xs text-gray-500 flex gap-1 items-center sm:text-sm dark:text-gray-400">
      <span class="i-carbon-information-square" />
      <span>输入您的Steam ID，获取AI生成的游戏库锐评</span>
    </div>
  </div>
</template>
