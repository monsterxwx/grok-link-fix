<template>
  <Teleport to="body">
    <div v-if="visible" class="fixed inset-0 z-[9999] flex items-center justify-center bg-black/50 backdrop-blur-sm transition-opacity" @click="handleBackdropClick">
      <div class="bg-white rounded-2xl p-6 max-w-sm w-full mx-4 relative shadow-2xl transition-transform" @click.stop>
        <!-- 标题区 -->
        <h3 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
          <!-- 自定义图标插槽 -->
          <slot name="icon">
             <div v-if="type === 'warning'" class="text-amber-500">
               <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
               </svg>
             </div>
             <div v-else-if="type === 'info'" class="text-purple-500">
               <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
               </svg>
             </div>
             <div v-else-if="type === 'error'" class="text-red-500">
               <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                 <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
               </svg>
             </div>
          </slot>
          {{ title }}
        </h3>
        
        <!-- 内容区 -->
        <div class="space-y-4">
          <slot>
            <p class="text-slate-600 text-sm leading-relaxed" v-html="content"></p>
          </slot>
          
          <!-- 操作区 -->
          <div class="flex justify-end gap-3 mt-6">
            <button
              v-if="showCancel"
              @click="handleCancel"
              class="px-4 py-2 bg-slate-100 text-slate-700 rounded-lg hover:bg-slate-200 transition-colors text-sm"
            >
              {{ cancelText }}
            </button>
            <button
              v-if="showConfirm"
              @click="handleConfirm"
              :class="confirmButtonClass"
              class="px-4 py-2 text-white rounded-lg transition-colors text-sm"
            >
              {{ confirmText }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  title: {
    type: String,
    default: '提示'
  },
  content: {
    type: String,
    default: ''
  },
  type: {
    type: String,
    default: 'info', // 'info' | 'warning' | 'error' | 'success' | 'primary'
  },
  showCancel: {
    type: Boolean,
    default: true
  },
  showConfirm: {
    type: Boolean,
    default: true
  },
  cancelText: {
    type: String,
    default: '取消'
  },
  confirmText: {
    type: String,
    default: '确定'
  },
  closeOnBackdrop: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['update:visible', 'confirm', 'cancel'])

const handleCancel = () => {
  emit('update:visible', false)
  emit('cancel')
}

const handleConfirm = () => {
  emit('confirm')
  // 注意：确定按钮默认不直接关闭弹窗，由父组件控制，应对异步提交等场景
}

const handleBackdropClick = () => {
  if (props.closeOnBackdrop) {
    handleCancel()
  }
}

// 根据 type 确定按钮颜色
const confirmButtonClass = computed(() => {
  switch (props.type) {
    case 'error':
    case 'warning':
      return 'bg-red-500 hover:bg-red-600'
    case 'primary':
      return 'bg-blue-500 hover:bg-blue-600'
    case 'info':
    case 'success':
    default:
      return 'bg-purple-500 hover:bg-purple-600'
  }
})
</script>
