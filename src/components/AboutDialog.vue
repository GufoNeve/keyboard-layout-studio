<template>
  <div
    v-if="show"
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    @click.self="$emit('cancel')"
  >
    <div class="bg-gray-800 rounded-lg p-6 max-w-md w-full mx-4 border border-gray-700">
      <div class="flex flex-col items-center gap-4">
        <!-- アイコン枠（未実装） -->
        <div class="w-24 h-24 bg-gray-700 rounded-lg border-2 border-gray-600 flex items-center justify-center">
          <span class="text-gray-500 text-sm">アイコン</span>
        </div>

        <!-- タイトル -->
        <h2 class="text-2xl font-bold text-white">Keyboard Layout Studio</h2>

        <!-- バージョン -->
        <div class="text-gray-300">
          <span class="text-sm">バージョン</span>
          <span class="ml-2 font-mono">{{ version }}</span>
        </div>

        <!-- GitHubリンク -->
        <a
          href="https://github.com/ymkn/keyboard-layout-studio/"
          target="_blank"
          rel="noopener noreferrer"
          class="text-blue-400 hover:text-blue-300 underline text-sm"
        >
          <i class="fa-brands fa-github"></i> GitHub
        </a>
      </div>

      <!-- 閉じるボタン -->
      <div class="mt-6 flex justify-end">
        <button
          @click="$emit('cancel')"
          class="px-4 py-2 bg-gray-700 text-white rounded hover:bg-gray-600 transition-colors"
        >
          閉じる
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Props {
  show: boolean
  version: string
}

defineProps<Props>()
defineEmits<{
  cancel: []
}>()
</script>
