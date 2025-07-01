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
      @delete="handleDelete"
      @page-change="handlePageChange"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PostForm from './PostForm.vue'
import PostList from './PostList.vue'

const posts = ref([])
const users = ref([])
const loading = ref(false)
const editingPost = ref(null)
const currentPage = ref(1)
const totalPosts = ref(0)
const itemsPerPage = ref(5)

const API_BASE = 'https://jsonplaceholder.typicode.com'

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
    alert('Ошибка при загрузке постов')
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
    alert('Пост успешно создан!')
  } catch (error) {
    console.error('Ошибка при создании поста:', error)
    alert('Ошибка при создании поста')
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

    alert('Пост успешно обновлен!')
  } catch (error) {
    console.error('Ошибка при обновлении поста:', error)
    alert('Ошибка при обновлении поста')
  } finally {
    loading.value = false
  }
}

const deletePost = async (id) => {
  if (!confirm('Вы уверены, что хотите удалить этот пост?')) {
    return
  }

  loading.value = true
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

    alert('Пост успешно удален!')
  } catch (error) {
    console.error('Ошибка при удалении поста:', error)
    alert('Ошибка при удалении поста')
  } finally {
    loading.value = false
  }
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
  deletePost(postId)
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
