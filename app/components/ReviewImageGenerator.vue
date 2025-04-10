<script setup lang="ts">
import html2canvas from 'html2canvas-pro'
import MarkdownIt from 'markdown-it'
import { computed, ref, watch } from 'vue'

const props = defineProps<{
  reviewText: string
  steamId: string
  isGeneratingImage: boolean
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
        <div
          ref="reviewTextRef"
          class="review-text markdown-content"
          :style="{ fontSize: `${fontSize}px` }"
          v-html="parsedReviewHtml"
        />
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

.steam-id {
  font-size: 18px;
  color: #666;
}
</style>
