<script setup lang="ts">
const route = useRoute()
const steamId = computed(() => route.query.steamId as string || '')
const dataId = computed(() => route.query.dataId as string || '')

// 如果没有steamId或dataId参数，重定向到首页
if (!steamId.value || !dataId.value) {
  navigateTo('/')
}
</script>

<template>
  <div class="py-6 h-full sm:py-10">
    <div class="mx-auto px-4 sm:px-6 container">
      <ReviewResult :steam-id="steamId" :data-id="dataId" />
    </div>
  </div>
</template>
