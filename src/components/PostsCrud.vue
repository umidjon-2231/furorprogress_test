<template>
  <div class="max-w-6xl mx-auto p-6">
    <h2 class="text-2xl font-bold text-gray-800 mb-6">Управление постами</h2>

    <PostForm
      :editing-post="editingPost"
      :loading="loading"
      @submit="handleFormSubmit"
      @cancel="handleFormCancel"
    />

    <PostList
      :posts="posts"
      :users="users"
      :loading="loading"
      :current-page="currentPage"
      :total-posts="totalPosts"
      :items-per-page="itemsPerPage"
      @edit="handleEdit"
      @delete="showDeleteModal"
      @page-change="handlePageChange"
    />

    <ConfirmModal
      :show="deleteModal.show"
      :loading="deleteModal.loading"
      title="Удаление поста"
      message="Вы уверены, что хотите удалить этот пост? Это действие нельзя отменить."
      confirm-text="Удалить"
      cancel-text="Отмена"
      @confirm="confirmDelete"
      @cancel="cancelDelete"
    />

    <Toast
      v-for="toast in toasts"
      :key="toast.id"
      :message="toast.message"
      :type="toast.type"
      :show="toast.show"
      @close="removeToast(toast.id)"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PostForm from './PostForm.vue'
import PostList from './PostList.vue'
import ConfirmModal from './ConfirmModal.vue'
import Toast from './Toast.vue'

const posts = ref([])
const users = ref([])
const loading = ref(false)
const editingPost = ref(null)
const currentPage = ref(1)
const totalPosts = ref(0)
const itemsPerPage = ref(5)

const deleteModal = ref({
  show: false,
  loading: false,
  postId: null
})

const toasts = ref([])
let toastIdCounter = 0

const API_BASE = 'https://jsonplaceholder.typicode.com'

const showToast = (message, type = 'info') => {
  const toast = {
    id: ++toastIdCounter,
    message,
    type,
    show: true
  }
  toasts.value.push(toast)
}

const removeToast = (id) => {
  const index = toasts.value.findIndex(toast => toast.id === id)
  if (index !== -1) {
    toasts.value.splice(index, 1)
  }
}

const fetchPosts = async (page = 1) => {
  loading.value = true
  try {
    const response = await fetch(`${API_BASE}/posts?_page=${page}&_limit=${itemsPerPage.value}`)
    posts.value = await response.json()

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
    showToast('Ошибка при загрузке постов', 'error')
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
    showToast('Ошибка при загрузке пользователей', 'error')
  }
}

const createPost = async (postData) => {
  loading.value = true
  try {
    const response = await fetch(`${API_BASE}/posts`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(postData)
    })
    const newPost = await response.json()

    newPost.id = Math.max(...posts.value.map(p => p.id), 0) + 1

    if (currentPage.value === 1) {
      posts.value.unshift(newPost)
      if (posts.value.length > itemsPerPage.value) {
        posts.value.pop()
      }
    }

    totalPosts.value++
    showToast('Пост успешно создан!', 'success')
  } catch (error) {
    console.error('Ошибка при создании поста:', error)
    showToast('Ошибка при создании поста', 'error')
  } finally {
    loading.value = false
  }
}

const updatePost = async (id, postData) => {
  loading.value = true
  try {
    const response = await fetch(`${API_BASE}/posts/${id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ ...postData, id })
    })
    const updatedPost = await response.json()

    const index = posts.value.findIndex(p => p.id === id)
    if (index !== -1) {
      posts.value[index] = updatedPost
    }

    showToast('Пост успешно обновлен!', 'success')
  } catch (error) {
    console.error('Ошибка при обновлении поста:', error)
    showToast('Ошибка при обновлении поста', 'error')
  } finally {
    loading.value = false
  }
}

const deletePost = async (id) => {
  deleteModal.value.loading = true
  try {
    await fetch(`${API_BASE}/posts/${id}`, {
      method: 'DELETE'
    })

    posts.value = posts.value.filter(p => p.id !== id)
    totalPosts.value--

    const totalPages = Math.ceil(totalPosts.value / itemsPerPage.value)
    if (currentPage.value > totalPages && totalPages > 0) {
      currentPage.value = totalPages
      await fetchPosts(currentPage.value)
    } else if (posts.value.length === 0 && totalPosts.value > 0) {
      await fetchPosts(currentPage.value)
    }

    showToast('Пост успешно удален!', 'success')
  } catch (error) {
    console.error('Ошибка при удалении поста:', error)
    showToast('Ошибка при удалении поста', 'error')
  } finally {
    deleteModal.value.loading = false
    deleteModal.value.show = false
    deleteModal.value.postId = null
  }
}

const showDeleteModal = (postId) => {
  deleteModal.value.show = true
  deleteModal.value.postId = postId
}

const confirmDelete = () => {
  if (deleteModal.value.postId) {
    deletePost(deleteModal.value.postId)
  }
}

const cancelDelete = () => {
  deleteModal.value.show = false
  deleteModal.value.postId = null
}

const handleFormSubmit = async (formData) => {
  if (editingPost.value) {
    await updatePost(editingPost.value.id, formData)
    editingPost.value = null
  } else {
    await createPost(formData)
  }
}

const handleFormCancel = () => {
  editingPost.value = null
}

const handleEdit = (post) => {
  editingPost.value = post
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const handleDelete = (postId) => {
  showDeleteModal(postId)
}

const handlePageChange = async (page) => {
  currentPage.value = page
  await fetchPosts(page)
}

onMounted(async () => {
  loading.value = true
  try {
    await Promise.all([fetchPosts(1), fetchUsers()])
  } finally {
    loading.value = false
  }
})
</script>
