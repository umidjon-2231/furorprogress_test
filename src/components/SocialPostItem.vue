<template>
  <div class="bg-white rounded-lg shadow-md">
    <!-- Post Header -->
    <div class="p-4 flex items-center space-x-3">
      <img
        :src="userAvatar"
        :alt="userName"
        class="w-10 h-10 rounded-full object-cover"
        @error="handleImageError"
      />
      <div class="flex-1">
        <h3 class="font-semibold text-gray-900">{{ userName }}</h3>
        <p class="text-sm text-gray-500">{{ timeAgo }} • 🌍</p>
      </div>
      <button class="text-gray-400 hover:text-gray-600">
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
          <path d="M10 6a2 2 0 110-4 2 2 0 010 4zM10 12a2 2 0 110-4 2 2 0 010 4zM10 18a2 2 0 110-4 2 2 0 010 4z"></path>
        </svg>
      </button>
    </div>

    <!-- Post Content -->
    <div class="px-4 pb-3">
      <h4 class="font-medium text-gray-900 mb-2">{{ post.title }}</h4>
      <p class="text-gray-700 leading-relaxed">{{ post.body }}</p>
    </div>

    <!-- Post Image (random placeholder) -->
    <div v-if="showImage" class="relative">
      <img
        :src="`https://picsum.photos/600/300?random=${post.id}`"
        :alt="post.title"
        class="w-full h-64 object-cover"
        @error="showImage = false"
      />
    </div>

    <!-- Post Actions -->
    <div class="px-4 py-3 border-t border-gray-100">
      <div class="flex items-center justify-between">
        <div class="flex items-center space-x-6">
          <button
            @click="toggleLike"
            :class="[
              'flex items-center space-x-2 text-sm transition-colors',
              isLiked ? 'text-red-500' : 'text-gray-500 hover:text-red-500'
            ]"
          >
            <svg class="w-5 h-5" :fill="isLiked ? 'currentColor' : 'none'" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"></path>
            </svg>
            <span>{{ likesCount }}</span>
          </button>

          <button
            @click="toggleComments"
            class="flex items-center space-x-2 text-sm text-gray-500 hover:text-blue-500 transition-colors"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"></path>
            </svg>
            <span>{{ commentsCount }}</span>
          </button>

          <button
            @click="$emit('share', post.id)"
            class="flex items-center space-x-2 text-sm text-gray-500 hover:text-green-500 transition-colors"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.367 2.684 3 3 0 00-5.367-2.684z"></path>
            </svg>
            <span>Поделиться</span>
          </button>
        </div>

        <button class="text-gray-400 hover:text-gray-600">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 5a2 2 0 012-2h10a2 2 0 012 2v16l-7-3.5L5 21V5z"></path>
          </svg>
        </button>
      </div>
    </div>

    <!-- Comments Preview -->
    <div v-if="showComments && commentsCount > 0" class="px-4 pb-4 border-t border-gray-100">
      <div class="space-y-2 mt-3">
        <div v-for="comment in props.comments.slice(0, 3)" :key="comment.id" class="flex items-start space-x-2">
          <img
            :src="`https://i.pravatar.cc/24?u=${comment.email}`"
            :alt="comment.name"
            class="w-6 h-6 rounded-full"
          />
          <div class="flex-1 bg-gray-100 rounded-lg px-3 py-2">
            <p class="text-sm">
              <span class="font-medium">{{ comment.name }}</span>
              {{ comment.body }}
            </p>
          </div>
        </div>

        <div v-if="props.comments.length > 3" class="text-center">
          <button
            @click="showAllComments = !showAllComments"
            class="text-sm text-blue-500 hover:text-blue-600"
          >
            {{ showAllComments ? 'Скрыть комментарии' : `Показать все ${props.comments.length} комментариев` }}
          </button>
        </div>

        <div v-if="showAllComments" class="space-y-2">
          <div v-for="comment in props.comments.slice(3)" :key="comment.id" class="flex items-start space-x-2">
            <img
              :src="`https://i.pravatar.cc/24?u=${comment.email}`"
              :alt="comment.name"
              class="w-6 h-6 rounded-full"
            />
            <div class="flex-1 bg-gray-100 rounded-lg px-3 py-2">
              <p class="text-sm">
                <span class="font-medium">{{ comment.name }}</span>
                {{ comment.body }}
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- Add Comment -->
      <div class="flex items-center space-x-2 mt-3">
        <img
          src="https://i.pravatar.cc/24?u=currentuser"
          alt="You"
          class="w-6 h-6 rounded-full"
        />
        <div class="flex-1 relative">
          <input
            v-model="newComment"
            type="text"
            placeholder="Написать комментарий..."
            class="w-full bg-gray-100 rounded-full px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
            @keyup.enter="addComment"
          />
        </div>
        <button
          @click="addComment"
          :disabled="!newComment.trim()"
          class="px-3 py-1 text-sm bg-blue-500 text-white rounded-full hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
        >
          Отправить
        </button>
      </div>
    </div>

    <!-- No comments state -->
    <div v-else-if="showComments && commentsCount === 0" class="px-4 pb-4 border-t border-gray-100">
      <div class="text-center py-4">
        <p class="text-sm text-gray-500">Пока нет комментариев</p>
      </div>

      <!-- Add Comment -->
      <div class="flex items-center space-x-2">
        <img
          src="https://i.pravatar.cc/24?u=currentuser"
          alt="You"
          class="w-6 h-6 rounded-full"
        />
        <div class="flex-1 relative">
          <input
            v-model="newComment"
            type="text"
            placeholder="Написать первый комментарий..."
            class="w-full bg-gray-100 rounded-full px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
            @keyup.enter="addComment"
          />
        </div>
        <button
          @click="addComment"
          :disabled="!newComment.trim()"
          class="px-3 py-1 text-sm bg-blue-500 text-white rounded-full hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
        >
          Отправить
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  post: {
    type: Object,
    required: true
  },
  user: {
    type: Object,
    default: null
  },
  comments: {
    type: Array,
    default: () => []
  }
})

defineEmits(['like', 'comment', 'share'])

const imageError = ref(false)
const isLiked = ref(false)
const likesCount = ref(Math.floor(Math.random() * 50) + 5)
const commentsCount = computed(() => props.comments.length)
const showImage = ref(Math.random() > 0.3)
const showComments = ref(false)
const showAllComments = ref(false)
const newComment = ref('')

const userName = computed(() => {
  return props.user ? props.user.name : `Пользователь ${props.post.userId}`
})

const userAvatar = computed(() => {
  if (imageError.value || !props.user) {
    return `https://ui-avatars.com/api/?name=${encodeURIComponent(userName.value)}&background=random&color=fff&size=40`
  }
  return `https://i.pravatar.cc/40?u=${props.user.email}`
})

const timeAgo = computed(() => {
  const hours = Math.floor(Math.random() * 24) + 1
  if (hours === 1) return '1 час назад'
  if (hours < 5) return `${hours} часа назад`
  return `${hours} часов назад`
})

const handleImageError = () => {
  imageError.value = true
}

const toggleLike = () => {
  isLiked.value = !isLiked.value
  likesCount.value += isLiked.value ? 1 : -1
}

const toggleComments = () => {
  showComments.value = !showComments.value
}

const addComment = () => {
  if (newComment.value.trim()) {
    console.log('Adding comment:', newComment.value)
    newComment.value = ''
  }
}
</script>
