<template>
  <div class="min-h-screen bg-gray-100">
    <!-- Header -->
    <header class="bg-white shadow-sm border-b sticky top-0 z-40">
      <div class="max-w-6xl mx-auto px-4">
        <div class="flex items-center justify-between h-16">
          <div class="flex items-center space-x-8">
            <h1 class="text-xl font-bold text-gray-900">SocialApp</h1>
            <nav class="hidden md:flex space-x-6">
              <button
                @click="activeTab = 'home'"
                :class="[
                  'px-3 py-2 text-sm font-medium transition-colors',
                  activeTab === 'home' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-600 hover:text-gray-900'
                ]"
              >
                Главная
              </button>
              <button
                @click="activeTab = 'manage'"
                :class="[
                  'px-3 py-2 text-sm font-medium transition-colors',
                  activeTab === 'manage' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-600 hover:text-gray-900'
                ]"
              >
                Управление
              </button>
            </nav>
          </div>
          <div class="flex items-center space-x-4">
            <div class="relative">
              <input
                type="text"
                placeholder="Поиск..."
                class="w-64 pl-10 pr-4 py-2 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              />
              <svg class="absolute left-3 top-2.5 h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
              </svg>
            </div>
            <div class="w-8 h-8 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full flex items-center justify-center">
              <span class="text-white font-semibold text-sm">U</span>
            </div>
          </div>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <div class="max-w-6xl mx-auto px-4 py-6">
      <!-- Home Tab -->
      <div v-if="activeTab === 'home'" class="grid grid-cols-1 lg:grid-cols-4 gap-6">
        <!-- Left Sidebar -->
        <div class="lg:col-span-1 space-y-4">
          <!-- User Profile Card -->
          <div class="bg-white rounded-lg shadow p-4">
            <div class="flex items-center space-x-3">
              <div class="w-12 h-12 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full flex items-center justify-center">
                <span class="text-white font-bold">U</span>
              </div>
              <div>
                <h3 class="font-semibold text-gray-900">Пользователь</h3>
                <p class="text-sm text-gray-600">Добро пожаловать!</p>
              </div>
            </div>
          </div>

          <!-- Quick Stats -->
          <div class="bg-white rounded-lg shadow p-4">
            <h3 class="font-semibold text-gray-900 mb-3">Статистика</h3>
            <div class="space-y-2">
              <div class="flex justify-between text-sm">
                <span class="text-gray-600">Всего постов</span>
                <span class="font-medium">{{ totalPosts }}</span>
              </div>
              <div class="flex justify-between text-sm">
                <span class="text-gray-600">Пользователей</span>
                <span class="font-medium">{{ users.length }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Main Feed -->
        <div class="lg:col-span-2 space-y-6">
          <!-- Create Post Card -->
          <div class="bg-white rounded-lg shadow p-4">
            <div class="flex items-center space-x-3">
              <div class="w-10 h-10 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full flex items-center justify-center">
                <span class="text-white font-medium text-sm">U</span>
              </div>
              <button
                @click="showCreatePost = true"
                class="flex-1 text-left px-4 py-2 bg-gray-100 rounded-full text-gray-500 hover:bg-gray-200 transition-colors"
              >
                Что у вас нового?
              </button>
            </div>
          </div>

          <!-- Posts Feed -->
          <div v-if="loading && !posts.length" class="text-center py-12">
            <div class="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500 mb-4"></div>
            <div class="text-gray-600">Загрузка ленты...</div>
          </div>

          <div v-else class="space-y-6">
            <SocialPostItem
              v-for="post in posts"
              :key="post.id"
              :post="post"
              :user="getUserById(post.userId)"
              :comments="getCommentsByPostId(post.id)"
              @like="handleLike"
              @comment="handleComment"
              @share="handleShare"
            />
          </div>

          <!-- Load More Button -->
          <div v-if="posts.length && !loading" class="text-center">
            <button
              @click="loadMorePosts"
              class="px-6 py-2 bg-blue-500 text-white rounded-full hover:bg-blue-600 transition-colors"
            >
              Загрузить еще
            </button>
          </div>
        </div>

        <!-- Right Sidebar -->
        <div class="lg:col-span-1 space-y-4">
          <!-- Trending -->
          <div class="bg-white rounded-lg shadow p-4">
            <h3 class="font-semibold text-gray-900 mb-3">Популярные темы</h3>
            <div class="space-y-2">
              <div class="text-sm">
                <p class="font-medium text-gray-900">#веб-разработка</p>
                <p class="text-gray-600">42 поста</p>
              </div>
              <div class="text-sm">
                <p class="font-medium text-gray-900">#vue</p>
                <p class="text-gray-600">28 постов</p>
              </div>
              <div class="text-sm">
                <p class="font-medium text-gray-900">#программирование</p>
                <p class="text-gray-600">35 постов</p>
              </div>
            </div>
          </div>

          <!-- Suggested Users -->
          <div class="bg-white rounded-lg shadow p-4">
            <h3 class="font-semibold text-gray-900 mb-3">Рекомендации</h3>
            <div class="space-y-3">
              <div v-for="user in users.slice(0, 3)" :key="user.id" class="flex items-center justify-between">
                <div class="flex items-center space-x-2">
                  <img
                    :src="`https://i.pravatar.cc/32?u=${user.email}`"
                    :alt="user.name"
                    class="w-8 h-8 rounded-full"
                  />
                  <div>
                    <p class="text-sm font-medium text-gray-900">{{ user.name }}</p>
                    <p class="text-xs text-gray-500">@{{ user.username }}</p>
                  </div>
                </div>
                <button class="text-xs bg-blue-500 text-white px-3 py-1 rounded-full hover:bg-blue-600">
                  Подписаться
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Manage Tab -->
      <div v-if="activeTab === 'manage'">
        <PostsCrud />
      </div>
    </div>

    <!-- Create Post Modal -->
    <CreatePostModal
      :show="showCreatePost"
      @close="showCreatePost = false"
      @created="handlePostCreated"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PostsCrud from './PostsCrud.vue'
import SocialPostItem from './SocialPostItem.vue'
import CreatePostModal from './CreatePostModal.vue'

const activeTab = ref('home')
const showCreatePost = ref(false)
const posts = ref([])
const users = ref([])
const comments = ref([])
const loading = ref(false)
const totalPosts = ref(0)
const currentPage = ref(1)

const API_BASE = 'https://jsonplaceholder.typicode.com'

const fetchPosts = async (page = 1, append = false) => {
  loading.value = true
  try {
    const response = await fetch(`${API_BASE}/posts?_page=${page}&_limit=5`)
    const newPosts = await response.json()

    if (append) {
      posts.value.push(...newPosts)
    } else {
      posts.value = newPosts
    }

    const totalCount = response.headers.get('x-total-count')
    if (totalCount) {
      totalPosts.value = parseInt(totalCount)
    } else {
      const allPostsResponse = await fetch(`${API_BASE}/posts`)
      const allPosts = await allPostsResponse.json()
      totalPosts.value = allPosts.length
    }
  } catch (error) {
    console.error('Ошибка при загрузке постов:', error)
  } finally {
    loading.value = false
  }
}

const fetchUsers = async () => {
  try {
    const response = await fetch(`${API_BASE}/users`)
    users.value = await response.json()
  } catch (error) {
    console.error('Ошибка при загрузке пользователей:', error)
  }
}

const fetchComments = async () => {
  try {
    const response = await fetch(`${API_BASE}/comments`)
    comments.value = await response.json()
  } catch (error) {
    console.error('Ошибка при загрузке комментариев:', error)
  }
}

const loadMorePosts = async () => {
  currentPage.value++
  await fetchPosts(currentPage.value, true)
}

const getUserById = (userId) => {
  return users.value.find(user => user.id === userId) || null
}

const getCommentsByPostId = (postId) => {
  return comments.value.filter(comment => comment.postId === postId)
}

const handleLike = (postId) => {
  console.log('Liked post:', postId)
}

const handleComment = (postId) => {
  console.log('Comment on post:', postId)
}

const handleShare = (postId) => {
  console.log('Share post:', postId)
}

const handlePostCreated = () => {
  showCreatePost.value = false
  fetchPosts(1)
}

onMounted(async () => {
  await Promise.all([fetchPosts(1), fetchUsers(), fetchComments()])
})
</script>
