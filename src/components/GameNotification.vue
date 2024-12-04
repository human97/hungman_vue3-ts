<script setup lang="ts">
import { ref } from 'vue'

const isVisible = ref<boolean>(false)

const open = (): void => {
  isVisible.value = true
}
const close = (): void => {
  isVisible.value = false
}

defineExpose({
  open,
  close
})
</script>

<template>
  <div class="notification-container" :class="{ show: isVisible }">
    <p>Вы уже вводили этот символ</p>
  </div>
</template>
