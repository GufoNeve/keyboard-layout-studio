<template>
  <div v-if="show" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-gray-800 rounded-lg shadow-xl max-w-2xl w-full mx-4 max-h-[80vh] flex flex-col">
      <div class="p-6 border-b border-gray-700">
        <h2 class="text-xl font-bold text-white">レイアウトを開く</h2>
      </div>

      <div class="flex-1 overflow-auto p-6">
        <div v-if="loading" class="text-center py-8">
          <div class="text-gray-400">読み込み中...</div>
        </div>

        <div v-else-if="error" class="text-center py-8">
          <div class="text-red-400">{{ error }}</div>
        </div>

        <div v-else-if="gists.length === 0" class="text-center py-8">
          <div class="text-gray-400">保存されたレイアウトがありません</div>
        </div>

        <div v-else class="space-y-2">
          <div
            v-for="item in gists"
            :key="item.id"
            @click="$emit('select', item)"
            class="bg-gray-700 rounded p-4 hover:bg-gray-600 transition-colors cursor-pointer"
          >
            <div class="flex items-start justify-between">
              <div class="flex-1 min-w-0">
                <h3 class="text-white font-semibold truncate">{{ item.filename }}</h3>
                <p v-if="item.description" class="text-gray-400 text-sm mt-1">
                  {{ item.description }}
                </p>
                <p class="text-gray-500 text-xs mt-2">
                  更新日時: {{ formatDate(item.updatedAt) }}
                </p>
              </div>
              <div class="flex gap-2 ml-4">
                <button
                  @click.stop="$emit('delete', item)"
                  class="px-3 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition-colors"
                  title="削除"
                >
                  <i class="fa-solid fa-trash-can"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="p-6 border-t border-gray-700 flex justify-end">
        <button
          @click="$emit('cancel')"
          class="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700 transition-colors"
        >
          キャンセル
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface SavedLayout {
  id: string
  filename: string
  description: string
  updatedAt: string
}

interface Props {
  show: boolean
  gists: SavedLayout[]
  loading: boolean
  error: string | null
}

interface Emits {
  (e: 'select', item: SavedLayout): void
  (e: 'delete', item: SavedLayout): void
  (e: 'cancel'): void
}

defineProps<Props>()
defineEmits<Emits>()

function formatDate(dateString: string): string {
  const date = new Date(dateString)
  return date.toLocaleString('ja-JP')
}
</script>
