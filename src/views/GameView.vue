<script setup lang="ts">
import GameHeader from '@/components/GameHeader.vue'
import GameFigure from '@/components/GameFigure.vue'
import GameWrongLetters from '@/components/GameWrongLetters.vue'
import GameWord from '@/components/GameWord.vue'
import GamePopup from '@/components/GamePopup.vue'
import GameNotification from '@/components/GameNotification.vue'

import { watch, computed, ref } from 'vue'

import { getRandomName } from '@/api/getRandomName'

const word = ref<string>('')
const letters = ref<string[]>([])
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const correctLetters = computed((): string[] =>
  letters.value.filter((letter: string): boolean => word.value.includes(letter))
)
const wrongLetters = computed((): string[] =>
  letters.value.filter((letter: string): boolean => !word.value.includes(letter))
)
const isLose = computed((): boolean => wrongLetters.value.length === 6)
const isWin = computed((): boolean =>
  [...word.value].every((letter: string): boolean => correctLetters.value.includes(letter))
)

const restart = async (): Promise<void> => {
  await getRandomWord()
  letters.value = []
  popup.value?.close()
}

const getRandomWord = async (): Promise<void> => {
  try {
    const name = await getRandomName()
    word.value = name.toUpperCase()
  } catch (error) {
    console.error(error)
    word.value = ''
  }
}

watch(wrongLetters, (): void => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})
watch(correctLetters, (): void => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isWin.value || isLose.value) return

  if (letters.value.includes(key.toUpperCase())) {
    notification.value?.open()
    setTimeout((): void | undefined => notification.value?.close(), 2000)
    return
  }
  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toUpperCase())
  }
})

getRandomWord()
</script>

<template>
  <GameHeader />

  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length" />
    <GameWrongLetters :wrong-letters="wrongLetters" />
    <GameWord :word="word" :correct-letters="correctLetters" />
  </div>

  <GamePopup ref="popup" :game-status="isWin ? 'win' : 'lose'" :word="word" @restart="restart" />
  <GameNotification ref="notification" />
</template>
