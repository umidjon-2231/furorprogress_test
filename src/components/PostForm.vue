<template>
  <div class="bg-white rounded-lg shadow-md p-6 mb-6">
    <h3 class="text-lg font-semibold mb-4">
      {{ editingPost ? 'Редактировать пост' : 'Добавить новый пост' }}
    </h3>
    <form @submit.prevent="handleSubmit" class="space-y-4">
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-2">Заголовок</label>
        <input
          v-model="form.title"
          type="text"
          required
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Введите заголовок поста"
        />
      </div>
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-2">Содержание</label>
        <textarea
          v-model="form.body"
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
          @click="handleCancel"
          class="px-4 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600"
        >
          Отмена
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  editingPost: {
    type: Object,
    default: null
  },
  loading: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['submit', 'cancel'])

const form = ref({
  title: '',
  body: '',
  userId: 1
})

const resetForm = () => {
  form.value = {
    title: '',
    body: '',
    userId: 1
  }
}

const handleSubmit = () => {
  emit('submit', { ...form.value })
  resetForm()
}

const handleCancel = () => {
  emit('cancel')
  resetForm()
}

watch(() => props.editingPost, (newPost) => {
  if (newPost) {
    form.value = {
      title: newPost.title,
      body: newPost.body,
      userId: newPost.userId
    }
  } else {
    resetForm()
  }
})
</script>
