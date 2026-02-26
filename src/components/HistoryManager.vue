<template>
  <div class="history-manager bg-white/80 backdrop-blur-xl rounded-3xl shadow-xl border border-white/50 p-6">
    <!-- 组件标题 -->
    <div class="flex justify-between items-center mb-1">
      <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
        <span class="w-1 h-6 bg-purple-500 rounded-full" />
        历史记录
      </h2>
      <div class="flex gap-2">
        <!-- 编辑模式切换按钮 -->
        <div
          @click="isManaging = !isManaging"
          class="cursor-pointer"
          :class="[
            'p-1.5 rounded-lg transition-colors',
            isManaging ? 'text-purple-600' : 'text-slate-400'
          ]"
          title="管理文件夹"
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
              d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"
            />
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
            />
          </svg>
        </div>
        <!-- 新建文件夹按钮 -->
        <button
          @click="showCreateFolderModal = true"
          class="px-3 py-1.5 bg-blue-500 hover:bg-blue-600 text-white rounded-lg text-sm transition-colors flex items-center gap-1"
        >
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
              d="M12 4v16m8-8H4"
            />
          </svg>
          新建文件夹
        </button>
      </div>
    </div>

    <!-- 文件夹树 -->
    <div class="mb-4">
      <div class="flex flex-wrap gap-2">
        <div
          v-for="folder in folders"
          :key="folder.id"
          @click="selectFolder(folder.id)"
          :class="[
            'flex items-center px-3 py-1.5 rounded-lg text-sm transition-all cursor-pointer max-w-full sm:max-w-[240px]',
            selectedFolderId === folder.id
              ? 'bg-purple-100 text-purple-700 border border-purple-200 font-medium'
              : 'bg-white text-slate-600 border border-slate-200 shadow-sm hover:border-purple-300 hover:text-purple-600'
          ]"
        >
          <div
            class="flex-1 text-left truncate"
            :title="folder.name"
          >
            <!-- 文件夹编辑状态 -->
            <input
              v-if="editingFolderId === folder.id"
              v-model="editFolderName"
              class="w-full text-sm font-bold text-slate-700 bg-white border border-purple-200 rounded px-1 focus:outline-none focus:ring-2 focus:ring-purple-500/20 focus:border-purple-500"
              @blur="saveFolderEdit(folder.id)"
              @keyup.enter="saveFolderEdit(folder.id)"
              @keyup.esc="cancelFolderEdit"
              :ref="el => { if(el) folderInputRefs[folder.id] = el }"
              @click.stop
            >
            <!-- 文件夹正常状态 -->
            <template v-else>
              <svg
                class="w-3 h-3 inline mr-1"
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
              <span class="align-middle">{{ folder.name }}</span>
            </template>
          </div>
          <div
            v-if="isManaging"
            class="pl-2 ml-1 border-l border-slate-200/50 flex items-center gap-1 shrink-0"
          >
            <!-- 编辑文件夹名称按钮 -->
            <div
              @click.stop="startFolderEdit(folder)"
              class="text-slate-400 hover:text-green-500 transition-colors p-1"
              title="重命名"
            >
              <svg
                 class="w-3.5 h-3.5"
                 fill="none"
                 stroke="currentColor"
                 viewBox="0 0 24 24"
              >
                 <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
              </svg>
            </div>
            <!-- 分享按钮 -->
            <div
              @click.stop="shareFolder(folder)"
              class="text-slate-400 hover:text-blue-500 transition-colors p-1"
              title="分享文件夹"
            >
              <svg
                class="w-3.5 h-3.5"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z"
                />
              </svg>
            </div>
            <!-- 删除按钮 -->
            <div
              @click.stop="showDeleteConfirm('folder', folder.id)"
              class="text-slate-400 hover:text-red-500 transition-colors p-1"
              title="删除文件夹"
            >
              <svg
                class="w-3.5 h-3.5"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                />
              </svg>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 历史记录列表 -->
    <div class="space-y-3 max-h-[400px] overflow-y-auto pr-2">
      <!-- 文件夹内链接 -->
      <div v-if="selectedFolderLinks.length === 0" class="text-center py-8 text-slate-400 text-sm">
        该文件夹为空
      </div>

      <!-- 链接列表项 -->
      <div
        v-for="link in displayLinks"
        :key="link.id"
        class="group relative p-3 rounded-2xl hover:bg-slate-50 transition-all border border-transparent hover:border-slate-100"
      >
        <div class="flex items-center gap-3">
          <!-- 图标容器 -->
          <div class="w-10 h-10 rounded-full bg-purple-50 flex items-center justify-center text-purple-500 shrink-0">
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
                d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"
              />
            </svg>
          </div>

          <!-- 内容区域 -->
          <div class="flex-1 min-w-0 w-0">
            <div class="mb-0.5">
              <!-- 标题/编辑框 -->
              <input
                v-if="editingId === link.id"
                v-model="editName"
                class="w-full text-sm font-bold text-slate-700 bg-white border border-purple-200 rounded px-2 py-0.5 focus:outline-none focus:ring-2 focus:ring-purple-500/20 focus:border-purple-500"
                @blur="saveEdit(link.id)"
                @keyup.enter="saveEdit(link.id)"
                @keyup.esc="cancelEdit"
                ref="editInput"
                @click.stop
              >
              <span
                v-else
                class="block text-sm font-bold text-slate-700 truncate cursor-pointer hover:text-purple-600 transition-colors"
                @click="startEdit(link)"
              >
                {{ link.name }}
              </span>
            </div>

            <!-- URL -->
            <a
              :href="link.url"
              target="_blank"
              rel="noopener noreferrer"
              class="text-xs text-slate-400 hover:text-purple-500 hover:underline truncate block transition-colors"
            >
              {{ link.url }}
            </a>

            <!-- 时间 -->
            <div class="text-[10px] text-slate-300 mt-0.5">
              {{ formatDate(link.createdAt) }}
            </div>
          </div>

          <!-- 操作按钮 (悬停显示，移动端常显) -->
          <div class="flex gap-1 shrink-0 opacity-100 sm:opacity-0 sm:group-hover:opacity-100 transition-opacity">
            <button
              @click.stop="showMoveMenu(link.id)"
              class="p-1.5 text-slate-400 hover:text-blue-500 hover:bg-blue-50 rounded-lg transition-colors"
              title="移动到文件夹"
            >
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
                  d="M8 7h12m0 0l-4-4m4 4l-4 4m0 6H4m0 0l4 4m-4-4l4-4"
                />
              </svg>
            </button>
            <button
              @click.stop="showDeleteConfirm('link', link.id)"
              class="p-1.5 text-slate-400 hover:text-red-500 hover:bg-red-50 rounded-lg transition-colors"
              title="删除"
            >
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
                  d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                />
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 新建文件夹弹窗 -->
    <Modal
      v-model:visible="showCreateFolderModal"
      title="新建文件夹"
      type="primary"
      confirm-text="创建"
      @confirm="createFolder"
    >
      <div>
        <label class="block text-sm font-medium text-slate-700 mb-1">文件夹名称</label>
        <input
          v-model="newFolderName"
          class="w-full p-3 border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-500"
          placeholder="输入文件夹名称"
          @keyup.enter="createFolder"
        >
      </div>
    </Modal>

    <!-- 删除确认弹窗 -->
    <!-- 删除确认弹窗 -->
    <Modal
      v-model:visible="deleteConfirmVisible"
      :title="cannotDeleteFolder ? '无法删除' : (deleteConfirmType === 'link' ? '删除链接' : '删除文件夹')"
      :type="cannotDeleteFolder ? 'warning' : 'error'"
      :show-cancel="!cannotDeleteFolder"
      :confirm-text="cannotDeleteFolder ? '知道了' : '删除'"
      @confirm="cannotDeleteFolder ? cancelDelete() : confirmDelete()"
      @cancel="cancelDelete"
    >
      <template #icon v-if="cannotDeleteFolder">
        <div class="text-amber-500">
           <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
           </svg>
        </div>
      </template>
      <div>
        <p v-if="cannotDeleteFolder" class="text-slate-600">
          该文件夹内包含链接，请先清空或移动链接后再尝试删除。
        </p>
        <p v-else class="text-slate-600">
          {{ deleteConfirmType === 'link' ? '确定要删除这个链接吗？' : '确定要删除这个文件夹吗？' }}
        </p>
      </div>
    </Modal>

    <!-- 移动链接弹窗 -->
    <!-- 移动链接弹窗 -->
    <Modal
      v-model:visible="showMoveModal"
      title="移动到文件夹"
      type="primary"
      :show-confirm="false"
      cancel-text="关闭"
    >
      <div>
        <label class="block text-sm font-medium text-slate-700 mb-2">选择目标文件夹</label>
        <div class="grid grid-cols-2 gap-2 max-h-[300px] overflow-y-auto">
          <button
            v-for="folder in folders"
            :key="folder.id"
            class="px-4 py-2 border border-slate-300 rounded-lg hover:bg-purple-500 hover:text-white transition-colors text-sm truncate"
            @click="moveToFolder(movingLinkId, folder.id)"
          >
            {{ folder.name }}
          </button>
        </div>
      </div>
    </Modal>
    
    <!-- 一般提示弹窗 -->
    <Modal
      v-model:visible="showAlertModal"
      :title="alertTitle"
      :content="alertContent"
      :show-cancel="false"
      type="info"
      confirm-text="知道了"
      @confirm="showAlertModal = false"
    />

    <!-- 分享成功提示 -->
    <Teleport to="body">
      <div
        v-if="showShareToast"
        class="fixed top-4 left-1/2 -translate-x-1/2 z-[9999] px-6 py-3 bg-slate-800 text-white rounded-full shadow-lg flex items-center gap-2 transition-all transform duration-300"
        :class="showShareToast ? 'translate-y-0 opacity-100' : '-translate-y-4 opacity-0'"
      >
        <svg class="w-5 h-5 text-green-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
        </svg>
        分享链接已复制到剪贴板！
      </div>
    </Teleport>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, nextTick } from 'vue'
import dayjs from 'dayjs'
import Modal from './Modal.vue'

// Props
const props = defineProps({
  // 从父组件传入的新链接
  newLink: {
    type: String,
    default: ''
  }
})

// Emits
const emit = defineEmits(['link-added'])

// 响应式数据
const links = ref([])
const folders = ref([])
const selectedFolderId = ref('root')
const editingId = ref(null)
const editName = ref('')
const showCreateFolderModal = ref(false)
const newFolderName = ref('')
const editInput = ref(null)
const movingLinkId = ref(null)
const showMoveModal = ref(false)
const isManaging = ref(false)
const deleteConfirmVisible = ref(false)
const deleteConfirmType = ref('link') // 'link' 或 'folder'
const deleteConfirmId = ref(null)
const cannotDeleteFolder = ref(false)

const showShareToast = ref(false)

// 文件夹编辑状态
const editingFolderId = ref(null)
const editFolderName = ref('')
const folderInputRefs = ref({})

// 一般提示弹窗状态
const showAlertModal = ref(false)
const alertTitle = ref('提示')
const alertContent = ref('')

const customAlert = (content, title = '提示') => {
  alertContent.value = content
  alertTitle.value = title
  showAlertModal.value = true
}

// 计算属性
const selectedFolderLinks = computed(() => {
  return links.value.filter(link => link.folderId === selectedFolderId.value)
})

const displayLinks = computed(() => {
  return selectedFolderLinks.value
})

// 方法
const loadFromStorage = () => {
  const savedLinks = localStorage.getItem('grokHistoryLinks')
  const savedFolders = localStorage.getItem('grokHistoryFolders')

  if (savedLinks) {
    links.value = JSON.parse(savedLinks)
  }

  if (savedFolders) {
    folders.value = JSON.parse(savedFolders)
  } else {
    // 默认创建一个文件夹
    folders.value = [
      { id: 'default', name: '默认文件夹', createdAt: new Date().toISOString() }
    ]
    saveToStorage()
  }
}

const saveToStorage = () => {
  localStorage.setItem('grokHistoryLinks', JSON.stringify(links.value))
  localStorage.setItem('grokHistoryFolders', JSON.stringify(folders.value))
}

const addLink = (url, name = '未命名') => {
  // 检查是否已存在相同的URL
  const existingLink = links.value.find(link => link.url === url)
  if (existingLink) {
    return
  }

  // 默认添加到第一个文件夹（默认文件夹）
  const defaultFolderId = folders.value.length > 0 ? folders.value[0].id : 'default'

  const newLink = {
    id: Date.now().toString() + Math.random().toString(36).substr(2, 9),
    name,
    url,
    folderId: defaultFolderId,
    createdAt: new Date().toISOString()
  }

  links.value.unshift(newLink) // 最新的放在前面
  saveToStorage()
  emit('link-added', newLink)
}

const startEdit = (link) => {
  editingId.value = link.id
  editName.value = link.name
  nextTick(() => {
    editInput.value?.focus()
    editInput.value?.select()
  })
}

const saveEdit = (id) => {
  const link = links.value.find(l => l.id === id)
  if (link && editName.value.trim()) {
    link.name = editName.value.trim()
    saveToStorage()
  }
  editingId.value = null
  editName.value = ''
}

const cancelEdit = () => {
  editingId.value = null
  editName.value = ''
}

// 文件夹重命名方法
const startFolderEdit = (folder) => {
  editingFolderId.value = folder.id
  editFolderName.value = folder.name
  nextTick(() => {
    const inputEl = folderInputRefs.value[folder.id]
    if (inputEl) {
      inputEl.focus()
      inputEl.select()
    }
  })
}

const saveFolderEdit = (id) => {
  const folder = folders.value.find(f => f.id === id)
  if (folder && editFolderName.value.trim()) {
    // 检查是否有重名 (除了自己)
    const newName = editFolderName.value.trim()
    const duplicate = folders.value.find(f => f.name === newName && f.id !== id)
    if (duplicate) {
      customAlert('已存在同名文件夹，请换个名字')
      return // 不清空状态，让用户继续编辑
    }
    folder.name = newName
    saveToStorage()
  }
  editingFolderId.value = null
  editFolderName.value = ''
}

const cancelFolderEdit = () => {
  editingFolderId.value = null
  editFolderName.value = ''
}

// 删除确认函数
const showDeleteConfirm = (type, id) => {
  deleteConfirmType.value = type
  deleteConfirmId.value = id

  if (type === 'folder') {
    const hasLinks = links.value.some(link => link.folderId === id)
    cannotDeleteFolder.value = hasLinks
  } else {
    cannotDeleteFolder.value = false
  }

  deleteConfirmVisible.value = true
}

const confirmDelete = () => {
  if (deleteConfirmType.value === 'link') {
    links.value = links.value.filter(link => link.id !== deleteConfirmId.value)
  } else if (deleteConfirmType.value === 'folder') {
    folders.value = folders.value.filter(folder => folder.id !== deleteConfirmId.value)
    // 如果删除的是当前选中的文件夹，切换到第一个文件夹
    if (selectedFolderId.value === deleteConfirmId.value && folders.value.length > 0) {
      selectedFolderId.value = folders.value[0].id
    }
  }
  saveToStorage()
  deleteConfirmVisible.value = false
  deleteConfirmId.value = null
  cannotDeleteFolder.value = false
}

const cancelDelete = () => {
  deleteConfirmVisible.value = false
  deleteConfirmId.value = null
}

const createFolder = () => {
  if (!newFolderName.value.trim()) return

  const folder = {
    id: Date.now().toString() + Math.random().toString(36).substr(2, 9),
    name: newFolderName.value.trim(),
    createdAt: new Date().toISOString()
  }

  folders.value.push(folder)
  saveToStorage()
  showCreateFolderModal.value = false
  newFolderName.value = ''
}

const selectFolder = (folderId) => {
  selectedFolderId.value = folderId
}

const showMoveMenu = (linkId) => {
  movingLinkId.value = linkId
  showMoveModal.value = true
}

const moveToFolder = (linkId, folderId) => {
  const link = links.value.find(l => l.id === linkId)
  if (link) {
    link.folderId = folderId
    saveToStorage()
    showMoveModal.value = false
    movingLinkId.value = null
  }
}

const cancelMove = () => {
  showMoveModal.value = false
  movingLinkId.value = null
}

const formatDate = (dateString) => {
  return dayjs(dateString).format('YYYY-MM-DD HH:mm')
}

// 分享文件夹逻辑
const shareFolder = async (folder) => {
  // 获取该文件夹下的所有链接
  const folderLinks = links.value.filter(link => link.folderId === folder.id)
  
  if (folderLinks.length === 0) {
    customAlert('该文件夹为空，无法分享')
    return
  }

  // 构建精简版分享数据对象
  const optimizedLinks = folderLinks.map(link => {
    let optimized = { u: link.url }
    
    // 如果名字不是"未命名"，才保留名字 (节省空间)
    if (link.name && link.name !== '未命名' && link.name !== '未命名链接') {
      optimized.n = link.name
    }
    
    // 尝试提取核心的 UUID (去掉前缀和后缀)
    const urlMatch = link.url.match(/post\/([a-fA-F0-9-]+)(\?|$)/)
    if (urlMatch && urlMatch[1]) {
      optimized.u = urlMatch[1] // 只存 UUID
    }
    
    return optimized
  })

  // 最精简的数据结构
  // [folderName, [ {n: name, u: uuid} ]]
  const shareData = [
    folder.name,
    optimizedLinks
  ]

  try {
    // 引入 lz-string 压缩数据
    // compressToEncodedURIComponent 可以生成非常短且 URL 安全的字符串
    const { compressToEncodedURIComponent } = await import('lz-string')
    
    const jsonStr = JSON.stringify(shareData)
    const encodedData = compressToEncodedURIComponent(jsonStr)
    
    // 构建分享链接
    const shareUrl = `${window.location.origin}${window.location.pathname}?s=${encodedData}`
    
    // 复制到剪贴板
    await navigator.clipboard.writeText(shareUrl)
    
    // 显示提示
    showShareToast.value = true
    setTimeout(() => {
      showShareToast.value = false
    }, 3000)
  } catch (err) {
    console.error('分享失败:', err)
    customAlert('生成分享链接失败，请重试')
  }
}

// 生命周期
onMounted(() => {
  loadFromStorage()
  // 默认选择第一个文件夹
  if (folders.value.length > 0) {
    selectedFolderId.value = folders.value[0].id
  }
})

// 监听新链接
watch(() => props.newLink, (newVal, oldVal) => {
  // 只有当newVal不为空且与oldVal不同时才添加
  if (newVal && newVal !== oldVal) {
    addLink(newVal)
  }
})

// 导入分享的文件夹
const importSharedFolder = (sharedData) => {
  if (!sharedData || !sharedData.folderName || !Array.isArray(sharedData.links)) {
    return false
  }

  // 1. 创建新文件夹
  const newFolderId = Date.now().toString() + Math.random().toString(36).substr(2, 9)
  
  // 检查是否有同名文件夹，如果有，在名字后面加上时间后缀
  let finalFolderName = sharedData.folderName
  const sameNameFolderExists = folders.value.some(f => f.name === finalFolderName)
  if (sameNameFolderExists) {
    finalFolderName = `${finalFolderName} (分享导入 ${dayjs().format('MM-DD HH:mm')})`
  }

  const newFolder = {
    id: newFolderId,
    name: finalFolderName,
    createdAt: new Date().toISOString()
  }
  folders.value.push(newFolder)

  // 2. 导入链接
  const newLinks = sharedData.links.map(link => {
    // 检查这个 URL 是否已经在本地存在
    // 如果想要允许重复链接存放在不同的文件夹，可以去掉这个检查
    // 这里我们简单起见，直接导入创建新的 ID
    return {
      id: Date.now().toString() + Math.random().toString(36).substr(2, 9),
      name: link.name || '未命名链接',
      url: link.url,
      folderId: newFolderId,
      createdAt: new Date().toISOString()
    }
  })

  // 将新链接加到列表最前面
  links.value = [...newLinks, ...links.value]
  
  // 3. 保存并选中新文件夹
  saveToStorage()
  selectedFolderId.value = newFolderId
  
  return true
}

// 暴露方法
defineExpose({
  addLink,
  importSharedFolder
})
</script>

<style scoped>
/* 滚动条样式 */
.space-y-3::-webkit-scrollbar {
  width: 6px;
}
.space-y-3::-webkit-scrollbar-track {
  border-radius: 3px;
  background: #f1f1f1;
}
.space-y-3::-webkit-scrollbar-thumb {
  border-radius: 3px;
  background: #c1c1c1;
}
.space-y-3::-webkit-scrollbar-thumb:hover {
  background: #a1a1a1;
}
</style>
