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
          <Transition name="adult-text" mode="out-in">
            <span :key="adultGamesText" class="text-white font-semibold text-xs inline-block origin-center">
              {{ adultGamesText }}
            </span>
          </Transition>
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
            <!-- 结果区域 -->
            <div class="flex-1 min-h-0 flex flex-col">
              <div class="flex justify-between items-center mb-2">
                <label class="block text-sm font-bold text-slate-700">解析结果 <span v-if="resultUrls.length > 0" class="text-xs text-purple-600 font-normal ml-2 bg-purple-100 px-2 py-0.5 rounded-full">共 {{ resultUrls.length }} 条</span></label>
                <div v-if="resultUrls.length > 0" class="flex gap-3">
                  <button @click="copyAllResult" class="text-xs text-purple-600 hover:text-purple-800 transition-colors flex items-center gap-1">
                    <span v-if="copied" class="text-green-600 flex items-center font-medium"><svg
                      class="w-3 h-3 mr-0.5"
                      fill="none"
                      stroke="currentColor"
                      viewBox="0 0 24 24"
                    ><path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M5 13l4 4L19 7"
                    /></svg>已复制全部</span>
                    <span v-else>复制全部</span>
                  </button>
                  <button @click="openAllUrl" class="text-xs text-pink-600 hover:text-pink-800 transition-colors">
                    打开全部
                  </button>
                </div>
              </div>

              <div v-if="resultUrls.length > 0" class="space-y-4 max-h-[350px] overflow-y-auto pr-2 custom-scrollbar">
                <div v-for="(url, idx) in resultUrls" :key="idx" class="relative group">
                  <!-- 结果显示框 -->
                  <div class="w-full p-4 pr-24 rounded-xl bg-purple-50 border border-purple-100 text-purple-900 break-all font-mono text-sm leading-relaxed">
                    {{ url }}
                  </div>

                  <!-- 悬浮/固定操作栏 -->
                  <div class="absolute right-2 top-2 bottom-2 flex flex-col justify-center gap-2">
                    <!-- 复制按钮 -->
                    <button
                      @click="copySingleResult(url, idx)"
                      class="flex items-center justify-center px-3 py-1.5 bg-white border border-purple-200 shadow-sm rounded-lg text-xs font-medium text-purple-700 hover:bg-purple-600 hover:text-white hover:border-purple-600 transition-all active:scale-95"
                    >
                      <span v-if="copiedIndex !== idx">复制</span>
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
                      </span>
                    </button>

                    <!-- 跳转按钮 -->
                    <button
                      @click="openSingleUrl(url)"
                      class="flex items-center justify-center px-3 py-1.5 bg-gradient-to-r from-purple-600 to-pink-600 shadow-lg shadow-purple-500/30 rounded-lg text-xs font-medium text-white hover:shadow-purple-500/50 hover:-translate-y-0.5 transition-all"
                    >
                      跳转
                    </button>
                  </div>
                </div>
              </div>

              <!-- 空状态/错误提示 -->
              <div v-else class="flex-1 min-h-[100px] flex items-center justify-center bg-slate-50 border border-dashed border-slate-200 rounded-xl text-slate-400 text-sm">
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
          <HistoryManager ref="historyManagerRef" :new-links="resultUrls" @link-added="handleLinkAdded" />
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

    <!-- 版本更新提示弹窗 -->
    <Modal
      v-model:visible="showUpdateModal"
      :title="updateTitle"
      type="primary"
      :show-cancel="false"
      confirm-text="我知道了"
      @confirm="confirmUpdate"
      @cancel="confirmUpdate"
      :close-on-backdrop="false"
    >
      <template #icon>
        <div class="text-purple-500 mt-5px">
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
              d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z"
            />
          </svg>
        </div>
      </template>

      <div v-html="updateContent" />
    </Modal>

    <!-- 导入分享文件夹弹窗 -->
    <Modal
      v-model:visible="showImportModal"
      title="发现分享的文件夹"
      type="primary"
      confirm-text="确定导入"
      @confirm="confirmImport"
      @cancel="cancelImport"
      :close-on-backdrop="false"
    >
      <template #icon>
        <div class="text-purple-500">
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
              d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"
            />
          </svg>
        </div>
      </template>

      <div class="space-y-4">
        <div class="p-4 bg-purple-50 rounded-xl border border-purple-100">
          <p class="font-bold text-purple-900 mb-1 flex items-center gap-2">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"
              />
            </svg>
            {{ sharedFolderData?.folderName }}
          </p>
          <p class="text-sm text-purple-700">
            包含 {{ sharedFolderData?.links?.length || 0 }} 个链接
          </p>
        </div>
        <p class="text-slate-600 text-sm">
          是否将该文件夹及其包含的链接保存到您的本地历史记录中？
        </p>
      </div>
    </Modal>

    <!-- 剪贴板检测弹窗 -->
    <Modal
      v-model:visible="showClipboardModal"
      title="检测到未处理的链接"
      type="primary"
      confirm-text="填入"
      cancel-text="忽略"
      @confirm="confirmClipboardFill"
      @cancel="cancelClipboardFill"
    >
      <template #icon>
        <div class="text-blue-500">
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
              d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4"
            />
          </svg>
        </div>
      </template>

      <div class="space-y-4">
        <p class="text-slate-600 text-sm">
          在剪贴板中检测到包含 UUID 的内容，是否自动填入并清空当前内容？
        </p>
        <div class="p-3 bg-slate-50 rounded-lg border border-slate-200 text-xs text-slate-500 break-all max-h-24 overflow-y-auto line-clamp-3">
          {{ clipboardContent }}
        </div>
      </div>
    </Modal>

    <!-- 全局群聊组件 -->
    <LiveChat />
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import HistoryManager from '@/components/HistoryManager.vue'
import Modal from '@/components/Modal.vue'
import LiveChat from '@/components/LiveChat.vue'

const inputText = ref('')
const copied = ref(false)
const historyManagerRef = ref(null)
const adultGamesText = ref('瑟瑟游戏')
let adultGamesTextTimer = null

// 分享导入相关的状态
const showImportModal = ref(false)
const sharedFolderData = ref(null)

// 剪贴板相关的状态
const showClipboardModal = ref(false)
const clipboardContent = ref('')
const lastCheckedClipboard = ref('')

// 版本更新提示相关
const showUpdateModal = ref(false)
const currentVersion = '1.4' // 👉 每次有新功能时，只需修改这个版本号
const updateTitle = '新功能发布 (v1.4)'
const updateContent = `
  <div class="space-y-2 text-sm text-slate-600">
    <p class="font-bold text-slate-800">本次更新内容：</p>
    <ul class="list-disc pl-5 space-y-1">
      <li>新增：支持解析多条链接啦，无论复制了多少条，都能一次性解析出来。再也不用自己一个个删除分割了。</li>
      <li>新增：全局匿名即时群聊功能，快来和大家畅所欲言吧！无任何限制，随便聊，随便分享好看的链接。</li>
      <li>新增：分享链接和文件夹功能，快来分享你的链接或文件夹吧！</li>
    </ul>
    <p class="mt-4 text-xs text-slate-400">（点击“我知道了”后，直到下次更新前不再打扰）</p>
  </div>
`

const checkUpdate = () => {
  const lastVersion = localStorage.getItem('grok_link_last_version')
  if (lastVersion !== currentVersion) {
    showUpdateModal.value = true
  }
}

const confirmUpdate = () => {
  localStorage.setItem('grok_link_last_version', currentVersion)
  showUpdateModal.value = false
}

// 监听输入，重置复制状态
watch(inputText, () => {
  copied.value = false
})

const clickAdultGames = () => {
  window.open('https://link3.cc/zyshare', '_blank')
}

// 核心解析逻辑
const resultUrls = computed(() => {
  if (!inputText.value) return []

  // 1. 第一步：核弹级清洗
  // 移除所有非 16 进制字符 (只保留 0-9, a-f, A-F)
  const rawHex = inputText.value.replace(/[^0-9a-fA-F]/g, '').toLowerCase()

  // 如果连32位都不够，肯定不是有效 UUID
  if (rawHex.length < 32) return []

  const urls = []
  let i = 0

  // 2. 第二步：滑动窗口特征扫描
  while (i <= rawHex.length - 32) {
    const segment = rawHex.substr(i, 32)

    const char12 = segment[12]
    const char16 = segment[16]

    let isMatch = false
    if (char12 === '4' && ['8', '9', 'a', 'b'].includes(char16)) {
      isMatch = true
    }

    if (isMatch) {
      const p1 = segment.substr(0, 8)
      const p2 = segment.substr(8, 4)
      const p3 = segment.substr(12, 4)
      const p4 = segment.substr(16, 4)
      const p5 = segment.substr(20, 12)
      const cleanUuid = `${p1}-${p2}-${p3}-${p4}-${p5}`
      urls.push(`https://grok.com/imagine/post/${cleanUuid}?source=copy_link&platform=android`)
      i += 32 // 匹配成功后跳过该段
    } else {
      i++ // 继续滑动
    }
  }

  // 兜底方案：如果一条都没找到，按照 32 位切割
  if (urls.length === 0) {
    let index = 0
    while (index <= rawHex.length - 32) {
      const segment = rawHex.substr(index, 32)
      const p1 = segment.substr(0, 8)
      const p2 = segment.substr(8, 4)
      const p3 = segment.substr(12, 4)
      const p4 = segment.substr(16, 4)
      const p5 = segment.substr(20, 12)
      const cleanUuid = `${p1}-${p2}-${p3}-${p4}-${p5}`
      urls.push(`https://grok.com/imagine/post/${cleanUuid}?source=copy_link&platform=android`)
      index += 32
    }
  }

  // 去重
  return [...new Set(urls)]
})

// 复制功能
const copiedIndex = ref(-1)

const copySingleResult = async (url, idx) => {
  try {
    await navigator.clipboard.writeText(url)
    copiedIndex.value = idx
    setTimeout(() => {
      if (copiedIndex.value === idx) copiedIndex.value = -1
    }, 2000)
  } catch (err) {
    console.error('Copy failed', err)
  }
}

const copyAllResult = async () => {
  if (!resultUrls.value.length) return
  try {
    const text = resultUrls.value.join('\n')
    await navigator.clipboard.writeText(text)
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch (err) {
    console.error('Copy all failed', err)
  }
}

// 处理链接添加事件
const handleLinkAdded = (link) => {
  console.log('Link added to history:', link)
}

// 跳转功能
const openSingleUrl = (url) => {
  if (url) {
    if (historyManagerRef.value) {
      historyManagerRef.value.addLink(url)
    }
    window.open(url, '_blank')
  }
}

const openAllUrl = () => {
  resultUrls.value.forEach(url => {
    if (historyManagerRef.value) {
      historyManagerRef.value.addLink(url)
    }
    window.open(url, '_blank')
  })
}

// 检查 URL 参数是否有分享数据
const checkShareData = async () => {
  const urlParams = new URLSearchParams(window.location.search)
  const shareDataParam = urlParams.get('shareData')
  const optimizedShareParam = urlParams.get('s')

  if (optimizedShareParam) {
    try {
      // 引入 lz-string 解压数据
      const { decompressFromEncodedURIComponent } = await import('lz-string')
      const decompressedStr = decompressFromEncodedURIComponent(optimizedShareParam)
      const parsedData = JSON.parse(decompressedStr)

      // 新版精简数据格式: [folderName, [ {n: name, u: uuid|url} ]]
      if (Array.isArray(parsedData) && parsedData.length === 2) {
        const folderName = parsedData[0]
        const rawLinks = parsedData[1]

        const reconstructedLinks = rawLinks.map(link => {
          let fullUrl = link.u
          // 如果只是单纯的 UUID，帮他拼回完整的 Grok URL
          if (/^[a-fA-F0-9-]{32,}$/.test(fullUrl)) {
            fullUrl = `https://grok.com/imagine/post/${fullUrl}?source=copy_link&platform=android`
          }
          return {
            name: link.n || '未命名',
            url: fullUrl
          }
        })

        sharedFolderData.value = {
          folderName,
          links: reconstructedLinks
        }
        showImportModal.value = true

        // 清除 URL 参数
        const newUrl = window.location.protocol + '//' + window.location.host + window.location.pathname
        window.history.replaceState({ path: newUrl }, '', newUrl)
      }
    } catch (err) {
      console.error('解析精简分享数据失败:', err)
    }
  } else if (shareDataParam) {
    // 兼容旧版的 base64 格式
    try {
      const decodedStr = decodeURIComponent(atob(shareDataParam))
      const parsedData = JSON.parse(decodedStr)

      if (parsedData && parsedData.folderName && Array.isArray(parsedData.links)) {
        sharedFolderData.value = parsedData
        showImportModal.value = true

        const newUrl = window.location.protocol + '//' + window.location.host + window.location.pathname
        window.history.replaceState({ path: newUrl }, '', newUrl)
      }
    } catch (err) {
      console.error('解析历史分享数据失败:', err)
    }
  }
}

// 取消导入
const cancelImport = () => {
  showImportModal.value = false
  sharedFolderData.value = null
}

// 确认导入
const confirmImport = () => {
  if (!sharedFolderData.value || !historyManagerRef.value) return

  // 调用暴露的 importSharedFolder 方法
  if (historyManagerRef.value.importSharedFolder) {
    historyManagerRef.value.importSharedFolder(sharedFolderData.value)
  }

  showImportModal.value = false
  sharedFolderData.value = null
}

// 检查剪贴板内容
const checkClipboard = async () => {
  // 当有其他弹窗打开时，不触发剪贴板检测，避免冲突
  if (showImportModal.value || showClipboardModal.value || showUpdateModal.value) return

  try {
    const text = await navigator.clipboard.readText()
    // 若为空、已检测过，或是当前输入框的内容，则跳过
    if (!text || text === lastCheckedClipboard.value || text === inputText.value) return

    // 基础校验：是否可能包含 UUID
    const rawHex = text.replace(/[^0-9a-fA-F]/g, '').toLowerCase()
    if (rawHex.length < 32) {
      lastCheckedClipboard.value = text
      return
    }

    let isValid = false
    for (let i = 0; i <= rawHex.length - 32; i++) {
      const segment = rawHex.substr(i, 32)
      if (segment[12] === '4' && ['8', '9', 'a', 'b'].includes(segment[16])) {
        isValid = true
        break
      }
    }
    // 兜底方案
    if (!isValid && rawHex.length >= 32) isValid = true

    if (isValid) {
      clipboardContent.value = text
      showClipboardModal.value = true
    } else {
      lastCheckedClipboard.value = text
    }
  } catch (err) {
    // 忽略读取失败的情况（例如用户未授权，或非安全上下文）
    console.debug('读取剪贴板失败，可能是权限未授予:', err)
  }
}

const confirmClipboardFill = () => {
  inputText.value = ''
  setTimeout(() => {
    inputText.value = clipboardContent.value
  }, 50)
  lastCheckedClipboard.value = clipboardContent.value
  showClipboardModal.value = false
}

const cancelClipboardFill = () => {
  lastCheckedClipboard.value = clipboardContent.value
  showClipboardModal.value = false
}

onMounted(() => {
  checkShareData()
  checkUpdate()
  adultGamesTextTimer = setInterval(() => {
    adultGamesText.value = adultGamesText.value === '瑟瑟游戏'
      ? '好用流量卡'
      : '瑟瑟游戏'
  }, 3000)
  window.addEventListener('focus', checkClipboard)
  // 初次加载也尝试检测一次
  setTimeout(checkClipboard, 500)
})

onUnmounted(() => {
  if (adultGamesTextTimer) {
    clearInterval(adultGamesTextTimer)
    adultGamesTextTimer = null
  }
  window.removeEventListener('focus', checkClipboard)
})
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

.adult-text-enter-active {
  transition: transform 520ms cubic-bezier(0.16, 1, 0.3, 1), opacity 520ms ease, filter 520ms ease, clip-path 520ms cubic-bezier(0.16, 1, 0.3, 1);
}

.adult-text-leave-active {
  transition: transform 280ms ease-in, opacity 280ms ease-in, filter 280ms ease-in;
}

.adult-text-enter-from {
  opacity: 0;
  transform: scale(0.35);
  filter: blur(8px);
  clip-path: circle(0% at 50% 50%);
}

.adult-text-enter-to {
  opacity: 1;
  transform: scale(1);
  filter: blur(0);
  clip-path: circle(150% at 50% 50%);
}

.adult-text-leave-from {
  opacity: 1;
  transform: scale(1);
  filter: blur(0);
}

.adult-text-leave-to {
  opacity: 0;
  transform: scale(0.2);
  filter: blur(4px);
}
</style>
