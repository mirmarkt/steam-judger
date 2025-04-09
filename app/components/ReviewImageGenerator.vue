<script setup lang="ts">
import html2canvas from 'html2canvas-pro'
import { ref } from 'vue'

const props = defineProps<{
  reviewText: string
  steamId: string
  isGeneratingImage: boolean
}>()

const emit = defineEmits<{
  'update:isGeneratingImage': [value: boolean]
}>()

const imageRef = ref<HTMLElement | null>(null)
const downloadLink = ref('')

// 生成图片
async function generateImage() {
  if (!imageRef.value)
    return

  try {
    emit('update:isGeneratingImage', true)

    // 使用html2canvas将DOM元素转换为canvas
    const canvas = await html2canvas(imageRef.value, {
      scale: 2, // 提高图片质量
      backgroundColor: '#ffffff',
      logging: false,
    })

    // 将canvas转换为图片URL
    const imageUrl = canvas.toDataURL('image/png')
    downloadLink.value = imageUrl

    // 自动下载图片
    const link = document.createElement('a')
    link.download = `steam-review-${props.steamId}.png`
    link.href = imageUrl
    link.click()

    emit('update:isGeneratingImage', false)
  }
  catch (error) {
    console.error('生成图片失败:', error)
    emit('update:isGeneratingImage', false)
  }
}

// 暴露方法给父组件
defineExpose({
  generateImage,
})
</script>

<template>
  <!-- 用于生成图片的模板 -->
  <div ref="imageRef" class="review-image-template">
    <div class="review-image-content">
      <!-- 头部 -->
      <div class="review-header">
        <div class="logo">
          <div class="logo-icon" />
          <div class="logo-text">
            <span class="logo-steam">Steam</span>
            <span class="logo-game">游戏</span>
            <span class="logo-review">锐评</span>
          </div>
        </div>
      </div>

      <!-- 锐评内容 -->
      <div class="review-body">
        <p class="review-text">
          {{ reviewText }}
        </p>
      </div>

      <!-- 底部 -->
      <div class="review-footer">
        <p class="steam-id">
          Steam ID: {{ steamId }}
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.review-image-template {
  width: 1080px;
  height: 1080px;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e8f0 100%);
  padding: 40px;
  box-sizing: border-box;
  position: fixed;
  left: -9999px; /* 隐藏元素，但仍然可以渲染 */
  top: -9999px;
  z-index: -1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.review-image-content {
  width: 100%;
  height: 100%;
  background-color: white;
  border-radius: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 40px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.review-header {
  margin-bottom: 40px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 16px;
}

.logo-icon {
  width: 60px;
  height: 60px;
  background-color: #4a89dc;
  border-radius: 12px;
  position: relative;
}

.logo-icon:before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 36px;
  height: 36px;
  background-color: white;
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Cpath d='M30 10V8h-2V6h-2V4h-2V2h-4v2h-2v2h-2v2h-2v2h-2v2H8v2H6v2H4v2H2v4h2v2h2v2h2v2h2v2h4v-2h2v-2h2v-2h2v-2h2v-2h2v-2h2v-2h2v-2h2v-4h-2zm-4 6h-2v2h-2v2h-2v2h-2v2h-2v2h-2v-2h-2v-2h-2v-2H8v-2H6v-2H4v-2h2v-2h2v-2h2v-2h2v-2h2v2h2v2h2v2h2v2h2v2h2v2z'/%3E%3C/svg%3E");
  mask-size: cover;
}

.logo-text {
  display: flex;
  align-items: center;
  font-size: 32px;
  font-weight: bold;
}

.logo-steam {
  color: #4a89dc;
  margin-right: 4px;
}

.logo-game {
  color: #333;
  margin-right: 4px;
}

.logo-review {
  background-color: #ffcc00;
  color: #333;
  padding: 2px 8px;
  border-radius: 4px;
}

.review-body {
  flex: 1;
  background-color: #f8f9fa;
  border-radius: 16px;
  padding: 30px;
  margin-bottom: 30px;
  overflow: hidden;
  position: relative;
}

.review-text {
  font-size: 24px;
  line-height: 1.6;
  color: #333;
  white-space: pre-line;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 12;
  -webkit-box-orient: vertical;
}

.review-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.steam-id {
  font-size: 18px;
  color: #666;
}
</style>
