<template>
  <div>
    <div v-if="loading && !posts.length" class="text-center py-12">
      <div class="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500 mb-4"></div>
      <div class="text-gray-600">Загрузка постов...</div>
    </div>

    <div v-else-if="posts.length" class="space-y-6">
      <div class="grid gap-4" :class="{ 'opacity-50 pointer-events-none': loading }">
        <PostItem
          v-for="post in posts"
          :key="post.id"
          :post="post"
          :user="getUserById(post.userId)"
          :loading="loading"
          @edit="handleEdit"
          @delete="handleDelete"
        />
      </div>

      <div v-if="loading" class="text-center py-4">
        <div class="inline-block animate-spin rounded-full h-6 w-6 border-b-2 border-blue-500 mr-2"></div>
        <span class="text-gray-600">Загрузка...</span>
      </div>

      <div v-if="totalPages > 1" class="flex justify-center items-center space-x-2">
        <button
          @click="goToPage(currentPage - 1)"
          :disabled="currentPage <= 1 || loading"
          class="px-3 py-2 text-sm bg-gray-200 text-gray-700 rounded hover:bg-gray-300 disabled:opacity-50 disabled:cursor-not-allowed transition-all"
        >
          Предыдущая
        </button>

        <div class="flex space-x-1">
          <button
            v-for="page in visiblePages"
            :key="page"
            @click="goToPage(page)"
            :disabled="loading"
            :class="[
              'px-3 py-2 text-sm rounded transition-all',
              page === currentPage
                ? 'bg-blue-500 text-white'
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300',
              loading && 'opacity-50 cursor-not-allowed'
            ]"
          >
            {{ page }}
          </button>
        </div>

        <button
          @click="goToPage(currentPage + 1)"
          :disabled="currentPage >= totalPages || loading"
          class="px-3 py-2 text-sm bg-gray-200 text-gray-700 rounded hover:bg-gray-300 disabled:opacity-50 disabled:cursor-not-allowed transition-all"
        >
          Следующая
        </button>
      </div>

      <div class="text-center text-sm text-gray-500">
        Показано {{ startItem }}-{{ endItem }} из {{ totalPosts }} постов
      </div>
    </div>

    <div v-else class="text-center py-8">
      <div class="text-gray-500">Посты не найдены</div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import PostItem from './PostItem.vue'

const props = defineProps({
  posts: {
    type: Array,
    default: () => []
  },
  users: {
    type: Array,
    default: () => []
  },
  loading: {
    type: Boolean,
    default: false
  },
  currentPage: {
    type: Number,
    default: 1
  },
  totalPosts: {
    type: Number,
    default: 0
  },
  itemsPerPage: {
    type: Number,
    default: 10
  }
})

const emit = defineEmits(['edit', 'delete', 'page-change'])

const totalPages = computed(() => {
  return Math.ceil(props.totalPosts / props.itemsPerPage)
})

const visiblePages = computed(() => {
  const total = totalPages.value
  const current = props.currentPage
  const pages = []

  if (total <= 7) {
    for (let i = 1; i <= total; i++) {
      pages.push(i)
    }
  } else {
    if (current <= 4) {
      for (let i = 1; i <= 5; i++) {
        pages.push(i)
      }
      pages.push('...')
      pages.push(total)
    } else if (current >= total - 3) {
      pages.push(1)
      pages.push('...')
      for (let i = total - 4; i <= total; i++) {
        pages.push(i)
      }
    } else {
      pages.push(1)
      pages.push('...')
      for (let i = current - 1; i <= current + 1; i++) {
        pages.push(i)
      }
      pages.push('...')
      pages.push(total)
    }
  }

  return pages.filter(page => page !== '...' || pages.indexOf(page) === pages.lastIndexOf(page))
})

const startItem = computed(() => {
  return (props.currentPage - 1) * props.itemsPerPage + 1
})

const endItem = computed(() => {
  const end = props.currentPage * props.itemsPerPage
  return end > props.totalPosts ? props.totalPosts : end
})

const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value && !props.loading) {
    emit('page-change', page)
  }
}

const handleEdit = (post) => {
  emit('edit', post)
}

const handleDelete = (postId) => {
  emit('delete', postId)
}

const getUserById = (userId) => {
  return props.users.find(user => user.id === userId) || null
}
</script>
