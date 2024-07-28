<script setup lang="ts">
import GameHeader from '@/components/GameHeader.vue'
import GameFigure from '@/components/GameFigure.vue'
import GameWrongLetters from '@/components/GameWrongLetters.vue'
import GameWord from '@/components/GameWord.vue'
import GamePopup from '@/components/GamePopup.vue'
import GameNotification from '@/components/GameNotification.vue'

import { watch, computed, ref } from 'vue'

import axios from 'axios'

const word = ref<string>('') // переделать при рефакторе
const letters = ref<string[]>([])
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const correctLetters = computed(() => letters.value.filter(letter => word.value.includes(letter)))
const wrongLetters = computed(() => letters.value.filter(letter => !word.value.includes(letter)))
const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every(letter => correctLetters.value.includes(letter)))

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})
watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isWin.value || isLose.value) return
  
  if (letters.value.includes(key.toUpperCase())) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return 
  }
  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toUpperCase())
  }
})

const restart = async () => {
  await getRandomName()
  letters.value = []
  popup.value?.close()
}

const getRandomName = async () => {
  try {
    const { data } = await axios.get<{ FirstName: string }>('https://api.randomdatatools.ru/?unescaped=false&params=FirstName')
    word.value = data.FirstName.toUpperCase()
  } catch (error) {
    console.error(error)
    word.value = ''
  } 
}

getRandomName()
</script>

<template>
  <GameHeader />

  <div class="game-container">
    <GameFigure
      :wrong-letters-count="wrongLetters.length" 
    />
    <GameWrongLetters 
      :wrong-letters="wrongLetters"
    />
    <GameWord 
      :word="word"
      :correct-letters="correctLetters" 
    /> 
  </div>
  
  <GamePopup
    ref="popup"
    :word="word"
    @restart="restart"
  />
  <GameNotification 
    ref="notification"
  /> 
</template>