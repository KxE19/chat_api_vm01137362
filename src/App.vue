<template>
  <div
    class="flex flex-col items-center justify-center w-screen min-h-screen bg-gray-100 text-gray-800 p-10"
  >
    <ChatApi />
  </div>
</template>

<script>
import ChatApi from './components/Chat_api.vue'

export default {
  components: {
    ChatApi
  }
}
</script>
