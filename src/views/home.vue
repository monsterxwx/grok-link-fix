<template>
  <div class="min-h-screen bg-slate-50 relative overflow-hidden font-sans selection:bg-purple-100">
    <!-- ================= 背景特效 (保持一致) ================= -->
    <div class="absolute inset-0 z-0 opacity-[0.4]" style="background-image: radial-gradient(#cbd5e1 1px, transparent 1px); background-size: 32px 32px;" />
    <div class="absolute top-0 right-[-10%] w-[500px] h-[500px] bg-purple-300 rounded-full mix-blend-multiply filter blur-[128px] opacity-40 animate-blob" />
    <div class="absolute bottom-[-10%] left-[-10%] w-[500px] h-[500px] bg-pink-300 rounded-full mix-blend-multiply filter blur-[128px] opacity-40 animate-blob animation-delay-2000" />

    <!-- 瑟瑟游戏按钮 -->
    <div
      @click="clickAdultGames"
      class="absolute top-6 right-6 z-50  cursor-pointer"
    >
      <div class="relative">
        <div class="absolute inset-0 bg-gradient-to-r from-purple-600 to-pink-600 rounded-lg blur opacity-40 group-hover:opacity-60 transition-all duration-300" />
        <div class="relative flex items-center gap-2 px-3 py-2 bg-gradient-to-r from-purple-600 to-pink-600 rounded-lg shadow-md shadow-purple-500/20 hover:shadow-purple-500/30 hover:-translate-y-0.5 transition-all duration-300">
          <span class="text-white font-semibold text-xs">瑟瑟游戏</span>
        </div>
      </div>
    </div>

    <!-- ================= 内容区域 ================= -->
    <div class="relative z-10 max-w-5xl mx-auto px-6 py-12">
      <!-- 头部 -->
      <div class="flex flex-col items-center mb-10 text-center">
        <h1 class="text-3xl md:text-4xl font-extrabold text-slate-900 mb-3">
          <span class="text-transparent bg-clip-text bg-gradient-to-r from-purple-600 to-pink-600">Grok 链接修复</span>
        </h1>
        <p class="text-slate-600">
          剔除乱码和表情，提取核心 UUID，生成可用的 Grok 生成的视频跳转链接。
        </p>
      </div>

      <!-- 主体 Grid -->
      <div class="grid lg:grid-cols-12 gap-8">
        <!-- 左侧：输入与结果区 (占 8 列) -->
        <div class="lg:col-span-8 space-y-6">
          <div class="bg-white/70 backdrop-blur-xl rounded-3xl shadow-xl border border-white/50 p-6 md:p-8 min-h-[400px] flex flex-col transition-all">
            <!-- 输入区域 -->
            <div class="mb-6">
              <label class="block text-sm font-bold text-slate-700 mb-2 flex justify-between">
                <span>原始文本</span>
                <span class="text-xs font-normal text-slate-400">支持粘贴任意包含 UUID 的乱码</span>
              </label>
              <textarea
                v-model="inputText"
                placeholder="粘贴你的内容，例如：看下我的8a3ae7bc-faaa...?source=..."
                class="w-full h-32 p-4 rounded-xl border border-slate-200 bg-white/50 focus:bg-white focus:border-purple-500 focus:ring-4 focus:ring-purple-500/10 transition-all outline-none resize-none text-slate-700 placeholder:text-slate-400"
              />
            </div>

            <!-- 转换按钮 (虽然自动转换，但提供按钮更有仪式感，也可用于清空) -->
            <div class="flex justify-end mb-6">
              <button
                v-if="inputText"
                @click="inputText = ''"
                class="text-sm text-slate-400 hover:text-slate-600 mr-4"
              >
                清空内容
              </button>
            </div>

            <!-- 结果区域 -->
            <div class="flex-1">
              <label class="block text-sm font-bold text-slate-700 mb-2">解析结果</label>

              <div v-if="resultUrl" class="relative group">
                <!-- 结果显示框 -->
                <div class="w-full p-4 pr-24 rounded-xl bg-purple-50 border border-purple-100 text-purple-900 break-all font-mono text-sm leading-relaxed">
                  {{ resultUrl }}
                </div>

                <!-- 悬浮/固定操作栏 -->
                <div class="absolute right-2 top-2 bottom-2 flex flex-col justify-center gap-2">
                  <!-- 复制按钮 -->
                  <button
                    @click="copyResult"
                    class="flex items-center justify-center px-3 py-1.5 bg-white border border-purple-200 shadow-sm rounded-lg text-xs font-medium text-purple-700 hover:bg-purple-600 hover:text-white hover:border-purple-600 transition-all active:scale-95"
                  >
                    <span v-if="!copied">复制</span>
                    <span v-else class="flex items-center gap-1">
                      <svg
                        class="w-3 h-3"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                      ><path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M5 13l4 4L19 7"
                      /></svg>
                      已复制
                    </span>
                  </button>

                  <!-- 跳转按钮 -->
                  <button
                    @click="openUrl"
                    class="flex items-center justify-center px-3 py-1.5 bg-gradient-to-r from-purple-600 to-pink-600 shadow-lg shadow-purple-500/30 rounded-lg text-xs font-medium text-white hover:shadow-purple-500/50 hover:-translate-y-0.5 transition-all"
                  >
                    跳转
                    <svg
                      class="w-3 h-3 ml-1"
                      fill="none"
                      stroke="currentColor"
                      viewBox="0 0 24 24"
                    ><path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h4m10 0h-4a2 2 0 00-2-2V8a2 2 0 002-2h4M6 6m0 0h12v12H6V6z"
                    /></svg>
                  </button>
                </div>
              </div>

              <!-- 空状态/错误提示 -->
              <div v-else class="h-full min-h-[100px] flex items-center justify-center bg-slate-50 border border-dashed border-slate-200 rounded-xl text-slate-400 text-sm">
                <span v-if="!inputText">等待输入...</span>
                <span v-else class="text-red-400 flex items-center gap-1">
                  <svg
                    class="w-4 h-4"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  ><path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                  /></svg>
                  未检测到有效的 UUID 格式
                </span>
              </div>
            </div>
          </div>
        </div>

        <!-- 右侧：说明区和历史记录 (占 4 列) -->
        <div class="lg:col-span-4 space-y-6">
          <!-- 历史记录组件 -->
          <HistoryManager ref="historyManagerRef" :new-link="resultUrl" @link-added="handleLinkAdded" />
          <!-- 捐赠区域 -->
          <div class="mt-8 pt-6 border-t border-slate-200">
            <div class="flex justify-center">
              <div
                class="inline-flex items-center gap-2 px-4 py-2 bg-gradient-to-r from-amber-100 to-orange-100 rounded-full border border-amber-200 cursor-pointer hover:from-amber-200 hover:to-orange-200 transition-all duration-300 group"
                @click="showDonationModal = true"
                @mouseenter="showDonationTooltip = true"
                @mouseleave="showDonationTooltip = false"
              >
                <span class="text-lg">☕️</span>
                <span class="text-sm font-medium text-amber-800">请作者喝杯咖啡</span>
                <svg
                  class="w-4 h-4 text-amber-600 group-hover:translate-x-1 transition-transform"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"
                  />
                </svg>
              </div>
            </div>
          </div>
          <div class="bg-white/80 backdrop-blur-xl rounded-3xl shadow-xl border border-white/50 p-6 sticky top-8">
            <h2 class="text-lg font-bold text-slate-800 mb-6 flex items-center gap-2">
              <span class="w-1 h-6 bg-purple-500 rounded-full" />
              工具说明
            </h2>

            <div class="space-y-4 text-sm text-slate-600 leading-relaxed">
              <p>
                本工具用于修复多余内容的 Grok 链接。
              </p>

              <div class="bg-blue-50 p-4 rounded-xl border border-blue-100">
                <p class="font-bold text-blue-800 mb-2">
                  识别规则：
                </p>
                <p>我们会扫描文本中符合 <code class="bg-white px-1 py-0.5 rounded border border-blue-200 text-blue-600 text-xs">8-4-4-4-12</code> 格式的 UUID 字符串。</p>
              </div>

              <div>
                <p class="font-medium text-slate-800 mb-2">
                  示例输入：
                </p>
                <div class="bg-slate-100 p-2 rounded text-xs text-slate-500 break-all">
                  看下我的8a3ae7bc-faaa-4100-aa82-13939d7ef2b8?source=copy_link
                </div>
              </div>

              <div class="flex justify-center pt-2">
                <svg
                  class="w-5 h-5 text-slate-400"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                ><path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M19 14l-7 7m0 0l-7-7m7 7V3"
                /></svg>
              </div>

              <div>
                <p class="font-medium text-slate-800 mb-2">
                  标准输出：
                </p>
                <div class="bg-purple-50 p-2 rounded text-xs text-purple-600 break-all border border-purple-100">
                  https://grok.com/imagine/post/8a3ae7bc-faaa-4100-aa82-13939d7ef2b8...
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 捐赠二维码弹窗 -->
    <div v-if="showDonationModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50 backdrop-blur-sm" @click="showDonationModal = false">
      <div class="bg-white rounded-2xl p-8 max-w-sm mx-4 relative shadow-2xl" @click.stop>
        <button
          @click="showDonationModal = false"
          class="absolute top-4 right-4 w-8 h-8 flex items-center justify-center text-gray-400 hover:text-gray-600 rounded-full hover:bg-gray-100 transition-all"
        >
          <svg
            class="w-5 h-5"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </button>

        <div class="text-center">
          <div class="text-3xl mb-2">
            ☕️
          </div>
          <h3 class="text-lg font-bold text-gray-800 mb-2">
            感谢您的支持！
          </h3>

          <!-- 二维码图片区域 -->
          <div class="bg-gray-50 rounded-xl  mb-4">
            <div class="w-48  mx-auto bg-white rounded-lg border-2 border-gray-200 flex items-center justify-center overflow-hidden">
              <img
                src="/juanzhen.jpg"
                alt="支付宝收款码"
                class="w-full h-full object-cover"
              >
            </div>
          </div>

          <p class="text-xs text-gray-400">
            您的支持是我继续开发的动力！
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import HistoryManager from '@/components/HistoryManager.vue'

const inputText = ref('')
const copied = ref(false)
const historyManagerRef = ref(null)
const showDonationModal = ref(false)
const showDonationTooltip = ref(false)

// 监听输入，重置复制状态
watch(inputText, () => {
  copied.value = false
})

const clickAdultGames = () => {
  window.open('https://link3.cc/zyshare', '_blank')
}

// 核心解析逻辑
const resultUrl = computed(() => {
  if (!inputText.value) return ''

  // 1. 第一步：核弹级清洗
  // 移除所有非 16 进制字符 (只保留 0-9, a-f, A-F)
  // 此时 "8a啊3ae7bc..." 会变成 "8a3ae7bc..."
  const rawHex = inputText.value.replace(/[^0-9a-fA-F]/g, '').toLowerCase()

  // 如果连32位都不够，肯定不是有效 UUID
  if (rawHex.length < 32) return ''

  // 2. 第二步：滑动窗口特征扫描
  // 也就是在乱码堆里找 "长得像 UUID v4" 的那一段
  let validUuid = ''

  // 遍历所有可能的 32 位片段
  for (let i = 0; i <= rawHex.length - 32; i++) {
    const segment = rawHex.substr(i, 32)

    // UUID v4 的特征校验：
    // 第 13 位 (index 12) 必须是 '4' (例如: ...-4xxx-...)
    // 第 17 位 (index 16) 必须是 '8', '9', 'a', 或 'b' (例如: ...-axxx-...)
    // 注意：Grok 目前生成的都是 v4 UUID，这个特征非常稳健
    const char12 = segment[12]
    const char16 = segment[16]

    if (char12 === '4' && ['8', '9', 'a', 'b'].includes(char16)) {
      validUuid = segment
      break // 找到了就停止，防止后面还有干扰
    }
  }

  // 兜底方案：如果没找到标准 v4 特征，但刚才清洗后的长度刚好或很长，
  // 可能是其他版本的 UUID，直接取前 32 位试一试
  if (!validUuid && rawHex.length >= 32) {
    validUuid = rawHex.substr(0, 32)
  }

  if (validUuid) {
    // 3. 重新格式化为 8-4-4-4-12
    const p1 = validUuid.substr(0, 8)
    const p2 = validUuid.substr(8, 4)
    const p3 = validUuid.substr(12, 4)
    const p4 = validUuid.substr(16, 4)
    const p5 = validUuid.substr(20, 12)

    const cleanUuid = `${p1}-${p2}-${p3}-${p4}-${p5}`
    return `https://grok.com/imagine/post/${cleanUuid}?source=copy_link&platform=android`
  }

  return ''
})

// 复制功能
const copyResult = async () => {
  if (!resultUrl.value) return
  try {
    await navigator.clipboard.writeText(resultUrl.value)
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch (err) {
    console.error('Copy failed', err)
  }
}

// 处理链接添加事件
const handleLinkAdded = (link) => {
  console.log('Link added to history:', link)
}

// 跳转功能
const openUrl = () => {
  if (resultUrl.value) {
    // 将链接添加到历史记录
    if (historyManagerRef.value) {
      historyManagerRef.value.addLink(resultUrl.value)
    }
    window.open(resultUrl.value, '_blank')
  }
}
</script>

<style scoped>
@keyframes blob {
  0% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(30px, -50px) scale(1.1); }
  66% { transform: translate(-20px, 20px) scale(0.9); }
  100% { transform: translate(0, 0) scale(1); }
}
.animate-blob { animation: blob 7s infinite; }
.animation-delay-2000 { animation-delay: 2s; }
</style>
