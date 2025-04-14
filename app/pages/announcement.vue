<script setup lang="ts">
import { appName } from '~/constants'
</script>

<template>
  <div class="px-4 py-8 flex flex-col items-center justify-center sm:px-6 sm:py-12">
    <div class="mb-6 text-center sm:mb-10">
      <div class="mb-3 flex justify-center sm:mb-4">
        <div class="i-carbon-warning-alt text-5xl text-yellow-500 sm:text-6xl" />
      </div>
      <h1 class="text-2xl font-bold mb-2 sm:text-3xl">
        <span class="text-blue-500">{{ appName }}</span>
        <span>维护公告</span>
      </h1>
      <p class="text-sm text-gray-500 mx-auto max-w-md sm:text-base dark:text-gray-400">
        感谢您一直以来对我们的支持与信任
      </p>
    </div>

    <div class="mb-6 p-6 rounded-xl bg-white max-w-md w-full shadow-lg sm:mb-8 sm:p-8 dark:bg-gray-800">
      <div class="flex flex-col">
        <div class="mb-6">
          <h2 class="text-lg font-medium mb-3 sm:text-xl">
            亲爱的朋友们：
          </h2>
          <div class="text-sm text-gray-700 space-y-4 dark:text-gray-300">
            <p>
              非常遗憾的宣布一件事，由于服务接口近期遭受了大量爆破请求，为了避免不必要的资源消耗和保护服务稳定性，吐槽姬不得不临时下线。
            </p>
            <p>
              作为个人，我的资源有限，这些异常访问已经导致API配额消耗过快，影响了正常用户的体验。我们深知这给您带来了不便，对此 深表歉意。
            </p>
            <p>
              目前，我正在积极寻找解决方案，感谢您一直以来的支持与理解，正是因为有你们在背后默默支持，我才有动力不断改进和维护这个服务。我会在服务恢复后第一时间通知您。
            </p>
          </div>
        </div>

        <div class="text-blue-700 p-4 rounded-lg bg-blue-50 flex gap-2 items-center dark:text-blue-300 dark:bg-blue-900/30">
          <span class="i-carbon-information text-lg flex-shrink-0" />
          <p class="text-sm">
            我们会通过各种渠道及时发布恢复通知，您也可以关注我们的GitHub仓库获取最新进展。
          </p>
        </div>
      </div>
    </div>

    <div class="flex flex-col gap-4 items-center">
      <NuxtLink to="/sponsor" class="text-blue-700 px-4 py-2 rounded-lg bg-blue-100 inline-flex gap-2 transition-colors items-center dark:text-blue-300 dark:bg-blue-900/50 hover:bg-blue-200 dark:hover:bg-blue-800/70">
        <span class="i-carbon-favorite" />
        <span>支持我们</span>
      </NuxtLink>

      <a href="https://github.com/kutius/steam-judger" target="_blank" class="text-gray-600 flex gap-1.5 transition-colors items-center dark:text-gray-300 hover:text-blue-500">
        <span class="i-carbon-logo-github" />
        <span>GitHub 仓库</span>
      </a>
    </div>
  </div>
</template>
