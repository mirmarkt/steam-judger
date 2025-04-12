<script setup lang="ts">
import { appName } from '~/constants'
</script>

<template>
  <div class="px-4 py-8 flex flex-col items-center justify-center sm:px-6 sm:py-12">
    <div class="mb-6 text-center sm:mb-10">
      <div class="mb-3 flex justify-center sm:mb-4">
        <div class="i-carbon-favorite text-5xl text-blue-500 sm:text-6xl" />
      </div>
      <h1 class="text-2xl font-bold mb-2 sm:text-3xl">
        <span>支持</span>
        <span class="text-blue-500 mx-1">{{ appName }}</span>
        <span>继续开发</span>
      </h1>
      <p class="text-sm text-gray-500 mx-auto max-w-md sm:text-base dark:text-gray-400">
        您的支持是我们持续改进的动力，感谢您的每一份贡献！
      </p>
    </div>

    <div class="mb-6 p-6 rounded-xl bg-white max-w-md w-full shadow-lg sm:mb-8 sm:p-8 dark:bg-gray-800">
      <div class="flex flex-col items-center">
        <div class="mb-4 text-center">
          <h2 class="text-lg font-medium mb-2 sm:text-xl">
            扫描二维码赞助我们
          </h2>
          <p class="text-sm text-gray-500 dark:text-gray-400">
            您的每一份支持都将用于服务器维护和功能开发
          </p>
        </div>

        <!-- 支付二维码 -->
        <div class="mb-4 flex flex-col gap-4 sm:flex-row sm:gap-6">
          <!-- 支付宝二维码 -->
          <div class="flex flex-col items-center">
            <div class="mb-2 border border-gray-200 rounded-lg overflow-hidden dark:border-gray-700">
              <img src="/sponsor/alipay.png" alt="支付宝赞助" class="h-40 w-40">
            </div>
            <span class="text-sm text-blue-500 font-medium">支付宝</span>
          </div>

          <!-- 微信支付二维码 -->
          <div class="flex flex-col items-center">
            <div class="mb-2 border border-gray-200 rounded-lg overflow-hidden dark:border-gray-700">
              <img src="/sponsor/wechat.png" alt="微信赞助" class="h-40 w-40">
            </div>
            <span class="text-sm text-green-500 font-medium">微信支付</span>
          </div>
        </div>

        <p class="text-sm text-gray-500 mt-4 text-center dark:text-gray-400">
          也可以通过
          <a href="mailto:precious.wei@qq.com" class="text-blue-500 hover:underline">联系我们</a>
          了解更多支持方式
        </p>
      </div>
    </div>

    <div class="text-xs text-gray-500 flex gap-1 items-center sm:text-sm dark:text-gray-400">
      <span class="i-carbon-favorite-filled" />
      <span>感谢您对
        <NuxtLink to="/" class="text-blue-500 hover:underline">{{ appName }}</NuxtLink>
        的支持和喜爱
      </span>
    </div>
  </div>
</template>
