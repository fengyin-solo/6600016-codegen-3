<template>
  <div class="flex flex-col gap-4">
    <div class="bg-gray-900 rounded-xl p-4">
      <h3 class="text-amber-300 font-bold mb-3">模拟电键输入</h3>
      <p class="text-gray-400 text-sm mb-4">
        按住鼠标或空格键发送信号，短按为点(.)，长按为划(-)。
        停顿自动分隔字符，更长停顿分隔单词。
      </p>

      <div class="flex flex-col items-center gap-4">
        <button
          @mousedown="handleKeyDown"
          @mouseup="handleKeyUp"
          @mouseleave="handleKeyUp"
          @touchstart.prevent="handleKeyDown"
          @touchend.prevent="handleKeyUp"
          class="w-40 h-40 rounded-full flex items-center justify-center text-xl font-bold transition-all select-none"
          :class="store.keyIsDown
            ? 'bg-amber-500 text-black scale-95 shadow-lg shadow-amber-500/50'
            : 'bg-gray-700 text-gray-300 hover:bg-gray-600'"
        >
          {{ store.keyIsDown ? '● 发送中' : '按住发送' }}
        </button>

        <div class="text-gray-400 text-sm">
          提示：也可以使用 <kbd class="px-2 py-1 bg-gray-700 rounded">空格键</kbd> 操作
        </div>
      </div>
    </div>

    <div class="grid grid-cols-2 gap-4">
      <div class="bg-gray-900 rounded-xl p-4">
        <h3 class="text-amber-300 font-bold mb-2">莫尔斯码</h3>
        <div class="w-full h-24 bg-gray-800 rounded p-3 overflow-y-auto text-green-400 font-mono text-lg">
          <span v-if="store.keyMorseOutput || store.keyCurrentCode">
            {{ store.keyMorseOutput }}<span v-if="store.keyCurrentCode" class="text-amber-400"> {{ store.keyCurrentCode }}_</span>
          </span>
          <span v-else class="text-gray-500">点击电键开始输入...</span>
        </div>
      </div>

      <div class="bg-gray-900 rounded-xl p-4">
        <h3 class="text-amber-300 font-bold mb-2">实时译文</h3>
        <div class="w-full h-24 bg-gray-800 rounded p-3 overflow-y-auto text-white text-lg">
          {{ store.keyDecodedText || '译文将显示在这里...' }}
          <span v-if="store.keyCurrentCode" class="inline-block w-2 h-5 bg-amber-400 animate-pulse ml-1"></span>
        </div>
      </div>
    </div>

    <div class="flex gap-2">
      <button @click="store.clearKeyInput()"
        class="bg-red-600 hover:bg-red-500 px-4 py-2 rounded text-white">
        清空输入
      </button>
    </div>

    <div class="bg-gray-900 rounded-xl p-4">
      <h3 class="text-amber-300 font-bold mb-2">输入速度指示</h3>
      <div class="flex items-center gap-2">
        <div class="flex-1 h-2 bg-gray-700 rounded-full overflow-hidden">
          <div class="h-full bg-amber-500 transition-all"
            :style="{ width: Math.min(100, (pressDuration / (store.dotDuration * 3)) * 100) + '%' }"></div>
        </div>
        <span class="text-gray-400 text-sm w-16 text-right">{{ Math.round(pressDuration) }}ms</span>
      </div>
      <div class="flex justify-between text-xs text-gray-500 mt-1">
        <span>点 ({{ store.dotDuration.toFixed(0) }}ms)</span>
        <span>划 ({{ (store.dotDuration * 3).toFixed(0) }}ms)</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { useMorseStore } from '../store/morse'

const store = useMorseStore()
const pressDuration = ref(0)
let pressInterval: number | null = null
let pressStart = 0

function handleKeyDown() {
  store.keyDown()
  pressStart = Date.now()
  pressDuration.value = 0
  if (pressInterval) clearInterval(pressInterval)
  pressInterval = window.setInterval(() => {
    pressDuration.value = Date.now() - pressStart
  }, 10)
}

function handleKeyUp() {
  store.keyUp()
  if (pressInterval) {
    clearInterval(pressInterval)
    pressInterval = null
  }
  setTimeout(() => {
    pressDuration.value = 0
  }, 200)
}

function handleKeyDownEvent(e: KeyboardEvent) {
  if (e.code === 'Space' && !e.repeat) {
    e.preventDefault()
    handleKeyDown()
  }
}

function handleKeyUpEvent(e: KeyboardEvent) {
  if (e.code === 'Space') {
    e.preventDefault()
    handleKeyUp()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyDownEvent)
  window.addEventListener('keyup', handleKeyUpEvent)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDownEvent)
  window.removeEventListener('keyup', handleKeyUpEvent)
  if (pressInterval) clearInterval(pressInterval)
})
</script>
