<script setup>
import PostsCrud from './components/PostsCrud.vue'
</script>

<template>
  <div id="app" class="min-h-screen bg-gradient-to-r from-blue-400 to-purple-500 py-8">
    <div class="container mx-auto px-4">
      <div class="bg-white rounded-lg shadow-xl p-8 max-w-7xl mx-auto">
        <h1 class="text-4xl font-bold text-gray-800 mb-8 text-center">
          CRUD приложение с JSONPlaceholder
        </h1>
        <PostsCrud />
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>
