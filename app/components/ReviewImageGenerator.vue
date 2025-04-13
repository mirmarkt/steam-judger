<script setup lang="ts">
import html2canvas from 'html2canvas-pro'
import MarkdownIt from 'markdown-it'
import { computed, ref, watch } from 'vue'

const props = defineProps<{
  reviewText: string
  steamId: string
  isGeneratingImage: boolean
  modelInfo?: { modelName: string, version: string } | null
  userInfo?: {
    steamId: string
    personaName: string
    profileUrl: string
    avatarIconUrl: string
    avatarMediumUrl: string
    avatarFullUrl: string
    personaState: number
    visibilityState: number
  } | null
}>()

const emit = defineEmits<{
  'update:isGeneratingImage': [value: boolean]
}>()

const imageRef = ref<HTMLElement | null>(null)
const reviewTextRef = ref<HTMLElement | null>(null)
const downloadLink = ref('')
const fontSize = ref(20) // 默认字体大小

// 初始化markdown-it解析器
const md = new MarkdownIt()

// 解析Markdown内容为HTML
const parsedReviewHtml = computed(() => {
  return props.reviewText ? md.render(props.reviewText) : ''
})

// 监听reviewText变化，调整字体大小
watch(() => props.reviewText, () => {
  // 在下一个渲染周期调整字体大小
  setTimeout(adjustFontSize, 0)
}, { immediate: true })

// 根据内容长度自动调整字体大小
function adjustFontSize() {
  if (!reviewTextRef.value || !imageRef.value)
    return

  // 获取容器元素
  const reviewBody = imageRef.value.querySelector('.review-body')
  if (!reviewBody)
    return

  // 初始字体大小
  fontSize.value = 20

  // 给内容一点时间渲染
  setTimeout(() => {
    // 获取容器高度和内容高度
    const containerHeight = reviewBody.clientHeight
    const contentHeight = reviewTextRef.value?.scrollHeight || 0

    // 如果内容高度超过容器高度，逐步减小字体大小
    if (contentHeight > containerHeight) {
      // 计算需要的缩放比例，更保守一点以确保完全显示
      const ratio = (containerHeight / contentHeight) * 0.9
      // 根据比例调整字体大小，但不小于10px
      const newSize = Math.max(10, Math.floor(fontSize.value * ratio))
      fontSize.value = newSize

      // 再次检查调整后的高度
      setTimeout(() => {
        const newContentHeight = reviewTextRef.value?.scrollHeight || 0
        if (newContentHeight > containerHeight) {
          // 如果仍然超出，再次调整，更激进地减小字体
          fontSize.value = Math.max(8, fontSize.value - 2)
        }
      }, 100)
    }
  }, 100)
}

// 生成图片
async function generateImage() {
  if (!imageRef.value)
    return

  try {
    emit('update:isGeneratingImage', true)

    // 确保在生成图片前调整字体大小并等待调整完成
    adjustFontSize()

    // 等待字体调整和渲染完成，增加等待时间确保完全渲染
    await new Promise(resolve => setTimeout(resolve, 500))

    // 再次检查并调整字体大小，确保内容完全适应容器
    const reviewBody = imageRef.value.querySelector('.review-body')
    const contentHeight = reviewTextRef.value?.scrollHeight || 0
    const containerHeight = reviewBody?.clientHeight || 0

    if (contentHeight > containerHeight) {
      // 如果内容仍然超出，进一步减小字体
      fontSize.value = Math.max(8, fontSize.value - 1)
      // 再次等待渲染
      await new Promise(resolve => setTimeout(resolve, 300))
    }

    // 使用html2canvas将DOM元素转换为canvas，增加配置以确保完整捕获
    const canvas = await html2canvas(imageRef.value, {
      scale: 2, // 提高图片质量
      backgroundColor: '#ffffff',
      logging: false,
      useCORS: true, // 允许跨域图片
      allowTaint: true, // 允许污染canvas
      onclone: (clonedDoc) => {
        // 在克隆的文档中确保内容可见
        const clonedElement = clonedDoc.querySelector('.review-text') as HTMLElement
        if (clonedElement) {
          clonedElement.style.overflow = 'visible'
          clonedElement.style.whiteSpace = 'normal'
        }
      },
    })

    // 将canvas转换为图片URL
    const imageUrl = canvas.toDataURL('image/png')
    downloadLink.value = imageUrl

    // 自动下载图片
    const link = document.createElement('a')
    // steam-review-xxx-0413.png
    const now = `${new Date().getMonth() + 1}${new Date().getDate()}`
    link.download = `steam-review-${props.userInfo?.personaName}-${now}.png`
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
        <div
          ref="reviewTextRef"
          class="review-text markdown-content"
          :style="{ fontSize: `${fontSize}px` }"
          v-html="parsedReviewHtml"
        />
      </div>

      <!-- 底部 -->
      <div class="review-footer">
        <div class="user-info">
          <!-- 用户头像 -->
          <div v-if="userInfo?.avatarMediumUrl" class="user-avatar">
            <img :src="userInfo.avatarMediumUrl" alt="Steam头像">
          </div>
          <div v-else class="user-avatar user-avatar-placeholder">
            <div class="user-avatar-icon" />
          </div>

          <!-- 用户ID信息 -->
          <div class="user-details">
            <p v-if="userInfo?.personaName" class="user-name">
              {{ userInfo.personaName }}
            </p>
            <p class="steam-id">
              Steam ID: {{ steamId }}
            </p>
          </div>
        </div>

        <p v-if="modelInfo" class="model-info">
          {{ modelInfo.modelName }} {{ modelInfo.version }}
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.review-image-template {
  width: 1080px;
  /* 使用自动高度，完全适应内容 */
  height: auto;
  min-height: 1080px;
  max-height: 3000px; /* 增加最大高度，确保长内容能够显示 */
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
  overflow: visible; /* 确保内容不被裁剪 */
}

.review-image-content {
  width: 100%;
  height: auto; /* 自动高度适应内容 */
  background-color: white;
  border-radius: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 40px;
  display: flex;
  flex-direction: column;
  overflow: visible; /* 确保内容不被裁剪 */
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
  /* 允许内容自然流动 */
  position: relative;
}

.review-text {
  /* 字体大小由JavaScript动态控制 */
  line-height: 1.6;
  color: #333;
  white-space: normal;
  /* 确保内容完整显示 */
  overflow: visible;
}

.review-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.user-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid #4a89dc;
}

.user-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.user-avatar-placeholder {
  background-color: #e4e8f0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.user-avatar-icon {
  width: 24px;
  height: 24px;
  background-color: #666;
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Cpath d='M16 8a5 5 0 1 0 5 5 5 5 0 0 0-5-5zm0 8a3 3 0 1 1 3-3 3 3 0 0 1-3 3z'/%3E%3Cpath d='M16 2a14 14 0 1 0 14 14A14 14 0 0 0 16 2zm0 26c-3.12 0-5.95-1.28-8-3.33v-2.89c0-2.08 3.5-3.33 8-3.33s8 1.25 8 3.33v2.89A11.91 11.91 0 0 1 16 28zm8-6.22c-1.81-1.4-4.54-2.11-8-2.11s-6.19.71-8 2.11v-1.11C8 19 11.5 18 16 18s8 1 8 2.67zm2-4.45A11.91 11.91 0 0 0 16 4a11.91 11.91 0 0 0-10 5.33v2.89c0-1.67 3.5-2.67 8-2.67s8 1 8 2.67z'/%3E%3C/svg%3E");
  mask-size: cover;
}

.user-details {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.user-name {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.steam-id {
  font-size: 14px;
  color: #666;
}

.model-info {
  font-size: 16px;
  color: #666;
  display: flex;
  align-items: center;
  gap: 4px;
}

.model-info:before {
  content: '';
  display: inline-block;
  width: 16px;
  height: 16px;
  background-color: #666;
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Cpath d='M16 2a14 14 0 1 0 14 14A14 14 0 0 0 16 2zm0 26a12 12 0 1 1 12-12 12 12 0 0 1-12 12z'/%3E%3Cpath d='M16 10a6 6 0 1 0 6 6 6 6 0 0 0-6-6zm0 10a4 4 0 1 1 4-4 4 4 0 0 1-4 4z'/%3E%3C/svg%3E");
  mask-size: cover;
}
</style>
