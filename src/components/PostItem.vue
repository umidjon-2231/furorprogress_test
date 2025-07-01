<template>
  <div class="relative flex w-full max-w-full flex-col rounded-xl bg-white bg-clip-border text-gray-700 shadow-md">
    <div class="relative mx-0 mt-4 flex items-center gap-4 overflow-hidden rounded-xl bg-transparent bg-clip-border pt-0 pb-4 px-6 text-gray-700 shadow-none">
      <img
        :src="userAvatar"
        :alt="userName"
        class="relative inline-block h-[58px] w-[58px] !rounded-full object-cover object-center"
        @error="handleImageError"
      />
      <div class="flex w-full flex-col gap-0.5">
        <div class="flex items-center justify-between">
          <h5 class="block font-sans text-xl font-semibold leading-snug tracking-normal text-gray-900 antialiased">
            {{ post.title }}
          </h5>
          <div class="flex items-center gap-2">
            <button
              @click="handleEdit"
              class="px-3 py-1 text-sm bg-yellow-500 text-white rounded hover:bg-yellow-600 transition-colors"
            >
              Редактировать
            </button>
            <button
              @click="handleDelete"
              :disabled="loading"
              class="px-3 py-1 text-sm bg-red-500 text-white rounded hover:bg-red-600 disabled:opacity-50 transition-colors"
            >
              Удалить
            </button>
          </div>
        </div>
        <p class="block font-sans text-base font-light leading-relaxed text-gray-600 antialiased">
          {{ userName }} • {{ userEmail }}
        </p>
      </div>
    </div>
    <div class="mb-6 px-6">
      <p class="block font-sans text-base font-light leading-relaxed text-gray-700 antialiased">
        {{ post.body }}
      </p>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  post: {
    type: Object,
    required: true
  },
  user: {
    type: Object,
    default: null
  },
  loading: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['edit', 'delete'])

const imageError = ref(false)

const userName = computed(() => {
  return props.user ? props.user.name : `Пользователь ${props.post.userId}`
})

const userEmail = computed(() => {
  return props.user ? props.user.email : `user${props.post.userId}@example.com`
})

const userAvatar = computed(() => {
  if (imageError.value || !props.user) {
    return `https://ui-avatars.com/api/?name=${encodeURIComponent(userName.value)}&background=random&color=fff&size=128`
  }
  return `https://i.pravatar.cc/128?u=${props.user.email}`
})

const handleImageError = () => {
  imageError.value = true
}

const handleEdit = () => {
  emit('edit', props.post)
}

const handleDelete = () => {
  emit('delete', props.post.id)
}
</script>
