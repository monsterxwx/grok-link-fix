<template>
  <div class="fixed bottom-6 right-6 z-[100] font-sans">
    <!-- 聊天主面板 -->
    <transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="transform translate-y-8 opacity-0"
      enter-to-class="transform translate-y-0 opacity-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="transform translate-y-0 opacity-100"
      leave-to-class="transform translate-y-8 opacity-0"
    >
      <div v-show="isOpen" class="absolute bottom-16 right-0 w-80 sm:w-96 bg-white/90 backdrop-blur-xl rounded-3xl shadow-2xl border border-purple-100 flex flex-col overflow-hidden transition-all h-[500px] max-h-[80vh]">
        <!-- 头部 -->
        <div class="px-5 py-4 bg-gradient-to-r from-purple-600 to-pink-600 text-white flex justify-between items-center shadow-lg">
          <div class="flex items-center gap-2">
            <span class="text-xl">💬</span>
            <div class="flex flex-col">
              <div class="flex items-center gap-2">
                <h3 class="font-bold text-sm tracking-wide">全球公共群聊室</h3>
                <span class="bg-white/20 px-1.5 py-0.5 rounded-full text-[10px] font-medium" v-if="isConnected">
                  {{ onlineCount }} 人在线
                </span>
              </div>
              <div class="flex items-center gap-1.5 text-[10px] opacity-90" :title="connectStatusText">
                <span class="w-1.5 h-1.5 rounded-full animate-pulse" :class="isConnected ? 'bg-green-400' : 'bg-yellow-400'"></span>
                {{ isConnected ? '实时在线' : '连接服务器中...' }}
              </div>
            </div>
          </div>
          <button @click="isOpen = false" class="text-white/80 hover:text-white bg-white/10 hover:bg-white/20 p-1.5 rounded-full transition-all active:scale-95">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
          </button>
        </div>

        <!-- 用户名设置区 -->
        <div class="px-5 py-2.5 border-b border-purple-50 bg-purple-50/50 flex items-center gap-2 text-xs text-purple-700/80">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>
          <span class="font-medium">当前身份:</span>
          <input
            v-if="isEditingName"
            v-model="tempName"
            @blur="saveName"
            @keyup.enter="saveName"
            ref="nameInput"
            class="flex-1 bg-white border border-purple-300 rounded-md px-2 py-1 outline-none focus:border-purple-600 focus:ring-2 focus:ring-purple-200 text-purple-900 font-bold shadow-sm"
            maxlength="15"
          />
          <div v-else class="flex-1 flex items-center gap-1.5 cursor-pointer group" @click="startEditName">
            <span class="font-bold text-purple-600 border-b border-dashed border-purple-300 group-hover:border-purple-600 truncate max-w-[120px]">{{ username }}</span>
            <svg class="w-3.5 h-3.5 text-purple-400 opacity-0 group-hover:opacity-100 transition-opacity" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg>
          </div>
        </div>

        <!-- 聊天记录区 -->
        <div class="flex-1 overflow-y-auto p-5 space-y-5 bg-slate-50/40 relative chat-scroll" ref="chatBox">
          <div v-if="messages.length === 0" class="absolute inset-0 flex flex-col items-center justify-center text-center px-4 pointer-events-none">
            <div class="w-16 h-16 bg-purple-100 rounded-full flex items-center justify-center mb-3">
              <span class="text-2xl">👋</span>
            </div>
            <p class="text-sm font-medium text-slate-500 mb-1">暂无新的群消息</p>
            <p class="text-xs text-slate-400">(公开免费信道，不保留过往历史记录)</p>
          </div>
          
          <div v-for="msg in messages" :key="msg.id" class="flex flex-col" :class="msg.isSelf ? 'items-end' : 'items-start'">
            <div class="text-[10px] text-slate-400 mb-1.5 px-1 flex items-center gap-1.5">
               <span v-if="!msg.isSelf" class="font-medium text-slate-500">{{ msg.user }}</span>
               <span :class="!msg.isSelf ? 'opacity-70' : ''">{{ msg.time }}</span>
            </div>
            <div 
              class="max-w-[85%] px-3.5 py-2.5 rounded-2xl text-sm break-words shadow-sm leading-relaxed"
              :class="msg.isSelf 
                  ? 'bg-gradient-to-br from-purple-500 to-pink-500 text-white rounded-tr-sm' 
                  : 'bg-white border border-slate-100/50 text-slate-700 rounded-tl-sm shadow-[0_2px_10px_rgba(0,0,0,0.03)]'"
            >
              {{ msg.text }}
            </div>
          </div>
        </div>

        <!-- 输入区 -->
        <div class="p-4 bg-white border-t border-purple-50">
          <form @submit.prevent="sendMessage" class="flex items-center gap-3">
            <input
              v-model="inputText"
              type="text"
              placeholder="说点什么一起畅所欲言..."
              class="flex-1 bg-slate-50 border border-slate-200 rounded-full px-5 py-2.5 text-sm outline-none focus:bg-white focus:border-purple-400 focus:ring-4 focus:ring-purple-100 transition-all text-slate-700 placeholder:text-slate-400"
              maxlength="150"
              :disabled="!isConnected"
            />
            <button
              type="submit"
              :disabled="!inputText.trim() || !isConnected"
              class="w-10 h-10 flex items-center justify-center rounded-full bg-gradient-to-r from-purple-600 to-pink-600 text-white shadow-lg hover:shadow-purple-500/40 hover:-translate-y-0.5 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:translate-y-0 disabled:shadow-none transition-all active:scale-95"
            >
               <svg class="w-4 h-4 ml-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"></path></svg>
            </button>
          </form>
        </div>
      </div>
    </transition>

    <!-- 悬浮按钮 -->
    <button
      @click="toggleChat"
      class="w-14 h-14 bg-gradient-to-r from-purple-600 to-pink-600 rounded-full shadow-xl shadow-purple-500/40 flex items-center justify-center text-white hover:-translate-y-1 hover:shadow-purple-500/60 transition-all relative z-50 group active:scale-95"
    >
      <!-- 有新消息小红点 -->
      <span v-if="hasUnread && !isOpen" class="absolute -top-1 -right-1 flex h-4 w-4">
        <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-red-400 opacity-75"></span>
        <span class="relative inline-flex rounded-full h-4 w-4 bg-red-500 border-2 border-white"></span>
      </span>
      
      <svg v-if="!isOpen" class="w-6 h-6 group-hover:scale-110 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"></path>
      </svg>
      <svg v-else class="w-6 h-6 group-hover:rotate-90 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24">
         <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
      </svg>
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick, computed } from 'vue'
import mqtt from 'mqtt'

// 状态
const isOpen = ref(false)
const hasUnread = ref(false)
const isConnected = ref(false)
const messages = ref([])
const inputText = ref('')

// 在线人数统计
const onlineUsers = ref(new Map()) // 记录 clientId -> lastSeen
const onlineCount = computed(() => onlineUsers.value.size || 1)
const heartbeatInterval = ref(null)

// 用户信息
const username = ref('匿名用户')
const isEditingName = ref(false)
const tempName = ref('')
const nameInput = ref(null)

// Client ID
const myClientId = 'chat_user_' + Math.random().toString(16).substr(2, 8)

// MQTT 客户端
let client = null
const topic = 'grok-link-fix-global-chat-v1-monsterxwx-public'
const systemTopic = 'grok-link-fix-global-chat-v1-system' // 用于心跳

// DOM 引用
const chatBox = ref(null)

const connectStatusText = computed(() => isConnected.value ? '已连接' : '连接服务器中...')

// 随机用户名生成
const generateRandomName = () => {
  const adjs = ['快乐的', '忧郁的', '神秘的', '奔跑的', '飞翔的', '勇敢的', '聪明的', '吃货', '调皮的', '可爱的']
  const nouns = ['猫咪', '狗狗', '小鸟', '熊猫', '老虎', '狮子', '海豚', '企鹅', '兔子', '仓鼠']
  return adjs[Math.floor(Math.random() * adjs.length)] + nouns[Math.floor(Math.random() * nouns.length)] + Math.floor(Math.random() * 1000)
}

// 初始化用户名
const initUser = () => {
  let savedName = localStorage.getItem('global_chat_username')
  if (!savedName) {
    savedName = generateRandomName()
    localStorage.setItem('global_chat_username', savedName)
  }
  username.value = savedName
}

// 修改用户名
const startEditName = () => {
  tempName.value = username.value
  isEditingName.value = true
  nextTick(() => {
    if (nameInput.value) nameInput.value.focus()
  })
}

const saveName = () => {
  if (tempName.value.trim()) {
    username.value = tempName.value.trim()
    localStorage.setItem('global_chat_username', username.value)
  }
  isEditingName.value = false
}

// MQTT 初始化
const initMqtt = () => {
  // 使用 EMQX 的公共免费 MQTT WebSocket 代理
  const brokerUrl = 'wss://broker.emqx.io:8084/mqtt'
  
  client = mqtt.connect(brokerUrl, {
    clientId: myClientId,
    clean: true,
    connectTimeout: 4000,
    reconnectPeriod: 2000,
  })

  client.on('connect', () => {
    isConnected.value = true
    client.subscribe(topic, { qos: 0 })
    client.subscribe(systemTopic, { qos: 0 })
    console.log('Public Chat Connected')
    
    // 启动心跳，每 5 秒广播一次自己在线
    startHeartbeat()
  })

  client.on('message', (t, message) => {
    if (t === systemTopic) {
      // 处理系统消息(心跳)
      try {
        const payload = JSON.parse(message.toString())
        if (payload.type === 'heartbeat' && payload.clientId) {
          onlineUsers.value.set(payload.clientId, Date.now())
          cleanOfflineUsers() // 清理离线用户
        }
      } catch (e) {
        // ignore
      }
      return
    }

    if (t === topic) {
      try {
        const payload = JSON.parse(message.toString())
        
        // 标记是否是自己发的消息
        payload.isSelf = payload.clientId === myClientId
        
        messages.value.push(payload)
        
        // 限制最多显示 100 条消息
        if (messages.value.length > 100) {
          messages.value.shift()
        }

        // 如果聊天窗没打开，且不是自己发的消息，则显示小红点
        if (!isOpen.value && !payload.isSelf) {
          hasUnread.value = true
        }

        scrollToBottom()
      } catch (e) {
        console.error('Failed to parse message', e)
      }
    }
  })

  client.on('close', () => {
    isConnected.value = false
    stopHeartbeat()
  })
  
  client.on('error', (err) => {
    console.error('MQTT error', err)
    isConnected.value = false
    stopHeartbeat()
  })
}

// 发关心跳包
const startHeartbeat = () => {
  stopHeartbeat()
  // 立即发送一次
  sendHeartbeat()
  heartbeatInterval.value = setInterval(sendHeartbeat, 5000)
}

const stopHeartbeat = () => {
  if (heartbeatInterval.value) {
    clearInterval(heartbeatInterval.value)
    heartbeatInterval.value = null
  }
}

const sendHeartbeat = () => {
  if (isConnected.value && client) {
    const payload = JSON.stringify({ type: 'heartbeat', clientId: myClientId })
    client.publish(systemTopic, payload, { qos: 0 })
    // 自己也要记录自己在线
    onlineUsers.value.set(myClientId, Date.now())
  }
}

// 清理超过 12 秒没有心跳的用户
const cleanOfflineUsers = () => {
  const now = Date.now()
  let hasDeleted = false
  for (const [id, lastSeen] of onlineUsers.value.entries()) {
    if (now - lastSeen > 12000) {
      onlineUsers.value.delete(id)
      hasDeleted = true
    }
  }
  // 强制触发响应式更新以防万一
  if (hasDeleted) {
    onlineUsers.value = new Map(onlineUsers.value)
  }
}

// 发送消息
const sendMessage = () => {
  if (!inputText.value.trim() || !isConnected.value) return

  const payload = {
    id: Date.now().toString() + Math.random().toString(16).substring(2, 6),
    user: username.value,
    text: inputText.value.trim(),
    time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' }),
    clientId: myClientId
  }

  // 通过 MQTT 发送
  client.publish(topic, JSON.stringify(payload))
  
  inputText.value = ''
  
  // 提示聚焦回输入框，方便连续发送
  nextTick(() => {
    scrollToBottom()
  })
}

// 切换聊天窗
const toggleChat = () => {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    hasUnread.value = false
    scrollToBottom()
  }
}

// 滚动到底部
const scrollToBottom = () => {
  nextTick(() => {
    if (chatBox.value) {
      // 添加了一点延迟和过渡，以适应发送后的 DOM 刷新
      setTimeout(() => {
        chatBox.value.scrollTo({
          top: chatBox.value.scrollHeight,
          behavior: 'smooth'
        })
      }, 50)
    }
  })
}

onMounted(() => {
  initUser()
  initMqtt()
})

onUnmounted(() => {
  if (client) {
    client.end()
  }
})
</script>

<style scoped>
/* 隐藏滚动条，但保留滚动功能 */
.chat-scroll::-webkit-scrollbar {
  width: 4px;
}
.chat-scroll::-webkit-scrollbar-track {
  background: transparent;
}
.chat-scroll::-webkit-scrollbar-thumb {
  background-color: rgba(167, 139, 250, 0.3);
  border-radius: 10px;
}
</style>
