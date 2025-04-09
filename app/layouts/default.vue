<template>
  <div class="text-gray-900 bg-gray-50 flex flex-col h-screen dark:text-gray-100 dark:bg-gray-900">
    <AppHeader />

    <main class="flex-1 w-full overflow-auto">
      <slot />
    </main>

    <AppFooter />
  </div>
</template>
