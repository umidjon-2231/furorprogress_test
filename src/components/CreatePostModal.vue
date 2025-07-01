<template>
  <div
    v-if="show"
    class="fixed inset-0 z-50 overflow-y-auto"
  >
    <div class="flex items-center justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
      <div
        class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity"
        @click="handleBackdropClick"
      ></div>

      <span class="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true">&#8203;</span>

      <div
        class="relative inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full z-10"
        @click.stop
      >
        <!-- Header -->
        <div class="bg-white px-4 pt-5 pb-4 sm:p-6 border-b border-gray-200">
          <div class="flex items-center justify-between">
            <h3 class="text-lg font-medium text-gray-900">
              Создать пост
            </h3>
            <button
              @click="close"
              class="text-gray-400 hover:text-gray-600"
            >
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          </div>
        </div>

        <!-- Content -->
        <div class="bg-white px-4 pt-4 pb-4 sm:p-6">
          <form @submit.prevent="handleSubmit">
            <!-- User Info -->
            <div class="flex items-center space-x-3 mb-4">
              <div class="w-10 h-10 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full flex items-center justify-center">
                <span class="text-white font-medium text-sm">U</span>
              </div>
              <div>
                <p class="font-medium text-gray-900">Пользователь</p>
                <p class="text-sm text-gray-500">Публичный пост</p>
              </div>
            </div>

            <!-- Title Input -->
            <div class="mb-4">
              <input
                v-model="form.title"
                type="text"
                required
                placeholder="Заголовок поста..."
                class="w-full border-0 resize-none text-lg placeholder-gray-400 focus:ring-0 focus:outline-none"
              />
            </div>

            <!-- Content Input -->
            <div class="mb-4">
              <textarea
                v-model="form.body"
                required
                rows="4"
                placeholder="О чем думаете?"
                class="w-full border-0 resize-none placeholder-gray-400 focus:ring-0 focus:outline-none"
              ></textarea>
            </div>

            <!-- Submit Button -->
            <button
              type="submit"
              :disabled="loading || !form.title.trim() || !form.body.trim()"
              class="w-full bg-blue-500 text-white py-2 px-4 rounded-lg hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
            >
              <span v-if="loading" class="inline-block animate-spin rounded-full h-4 w-4 border-b-2 border-white mr-2"></span>
              {{ loading ? 'Публикация...' : 'Опубликовать' }}
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  show: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close', 'created'])

const loading = ref(false)
const form = ref({
  title: '',
  body: '',
  userId: 1
})

const close = () => {
  emit('close')
  resetForm()
}

const handleBackdropClick = () => {
  if (!loading.value) {
    close()
  }
}

const resetForm = () => {
  form.value = {
    title: '',
    body: '',
    userId: 1
  }
}

const handleSubmit = async () => {
  loading.value = true
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(form.value)
    })

    if (response.ok) {
      emit('created')
      resetForm()
    }
  } catch (error) {
    console.error('Ошибка при создании поста:', error)
  } finally {
    loading.value = false
  }
}
</script>
