<template>
  <div class="max-w-6xl mx-auto p-6">
    <h2 class="text-2xl font-bold text-gray-800 mb-6">Управление постами</h2>

    <div class="bg-white rounded-lg shadow-md p-6 mb-6">
      <h3 class="text-lg font-semibold mb-4">
        {{ editingPost ? 'Редактировать пост' : 'Добавить новый пост' }}
      </h3>
      <form @submit.prevent="savePost" class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-2">Заголовок</label>
          <input
            v-model="postForm.title"
            type="text"
            required
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Введите заголовок поста"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-2">Содержание</label>
          <textarea
            v-model="postForm.body"
            required
            rows="4"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Введите содержание поста"
          ></textarea>
        </div>
        <div class="flex gap-2">
          <button
            type="submit"
            :disabled="loading"
            class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 disabled:opacity-50"
          >
            {{ loading ? 'Сохранение...' : (editingPost ? 'Обновить' : 'Создать') }}
          </button>
          <button
            v-if="editingPost"
            type="button"
            @click="cancelEdit"
            class="px-4 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600"
          >
            Отмена
          </button>
        </div>
      </form>
    </div>

    <div v-if="loading && !posts.length" class="text-center py-8">
      <div class="text-gray-600">Загрузка постов...</div>
    </div>

    <div v-else class="grid gap-4">
      <div
        v-for="post in posts"
        :key="post.id"
        class="bg-white rounded-lg shadow-md p-6 border border-gray-200"
      >
        <div class="flex justify-between items-start mb-3">
          <h3 class="text-lg font-semibold text-gray-800 flex-1">{{ post.title }}</h3>
          <div class="flex gap-2 ml-4">
            <button
              @click="editPost(post)"
              class="px-3 py-1 text-sm bg-yellow-500 text-white rounded hover:bg-yellow-600"
            >
              Редактировать
            </button>
            <button
              @click="deletePost(post.id)"
              :disabled="loading"
              class="px-3 py-1 text-sm bg-red-500 text-white rounded hover:bg-red-600 disabled:opacity-50"
            >
              Удалить
            </button>
          </div>
        </div>
        <p class="text-gray-600 text-sm mb-2">ID: {{ post.id }} | Пользователь: {{ post.userId }}</p>
        <p class="text-gray-700">{{ post.body }}</p>
      </div>
    </div>

    <div v-if="!loading && posts.length === 0" class="text-center py-8">
      <div class="text-gray-500">Посты не найдены</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const posts = ref([])
const loading = ref(false)
const editingPost = ref(null)
const postForm = ref({
  title: '',
  body: '',
  userId: 1
})

const API_BASE = 'https://jsonplaceholder.typicode.com'

const fetchPosts = async () => {
  loading.value = true
  try {
    const response = await fetch(`${API_BASE}/posts`)
    posts.value = await response.json()
  } catch (error) {
    console.error('Ошибка при загрузке постов:', error)
    alert('Ошибка при загрузке постов')
  } finally {
    loading.value = false
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
    posts.value.unshift(newPost)

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
    alert('Пост успешно удален!')
  } catch (error) {
    console.error('Ошибка при удалении поста:', error)
    alert('Ошибка при удалении поста')
  } finally {
    loading.value = false
  }
}

const savePost = async () => {
  if (editingPost.value) {
    await updatePost(editingPost.value.id, postForm.value)
    editingPost.value = null
  } else {
    await createPost(postForm.value)
  }

  postForm.value = {
    title: '',
    body: '',
    userId: 1
  }
}

const editPost = (post) => {
  editingPost.value = post
  postForm.value = {
    title: post.title,
    body: post.body,
    userId: post.userId
  }

  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const cancelEdit = () => {
  editingPost.value = null
  postForm.value = {
    title: '',
    body: '',
    userId: 1
  }
}

onMounted(() => {
  fetchPosts()
})
</script>
