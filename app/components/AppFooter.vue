<template>
  <footer class="mt-auto py-4 sm:py-6">
    <div class="mx-auto px-4 container">
      <div class="text-xs text-gray-500 flex flex-col items-center justify-between sm:text-sm dark:text-gray-400 md:flex-row">
        <div class="mb-4 flex gap-1 items-center md:mb-0">
          <span class="i-carbon-game-console" />
          <span class="text-center md:text-left">© {{ new Date().getFullYear() }} Steam 游戏库吐槽姬 - AI生成的游戏库分析</span>
        </div>

        <div class="flex flex-wrap gap-3 items-center justify-center sm:gap-4">
          <a href="#" class="flex gap-1 transition-colors items-center hover:text-blue-500">
            <span class="i-carbon-help" />
            <span>帮助</span>
          </a>  
          <a href="https://github.com" target="_blank" class="flex gap-1 transition-colors items-center hover:text-blue-500">
            <span class="i-carbon-logo-github" />
            <span>GitHub</span>
          </a>
        </div>
      </div>
    </div>
  </footer>
</template>
