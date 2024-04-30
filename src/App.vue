<script setup lang="ts">
import { ref, computed, watch } from 'vue'

import GameHeader from './assets/components/GameHeader.vue'
import GameFigure from './assets/components/GameFigure.vue'
import GameWrongLetters from './assets/components/GameWrongLetters.vue'
import GameWord from './assets/components/GameWord.vue'
import GamePopup from './assets/components/GamePopup.vue'
import GameNotification from './assets/components/GameNotification.vue'
import { getRandomName } from './api/getRandomName'
import { useRandomWord } from './composables/useRandomWord'
import { useLetters } from './composables/useLetters'
import { useNotification } from './composables/useNotification'

const { word, getRandomWord } = useRandomWord()

const { letters, correctLetters, wrongLetters, isLose, isWin, addLetter, resetLetters } =
  useLetters(word)

const { notification, showNotification } = useNotification()
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const restart = async () => {
  await getRandomWord()

  resetLetters()
  popup.value?.close()
}

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
  if (isLose.value || isWin.value) return
  if (letters.value.includes(key)) {
    showNotification()
  }
  addLetter(key)
})
</script>
<template>
  <GameHeader />
  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length" />
    <GameWrongLetters :wrong-letters="wrongLetters" />

    <GameWord :word="word" :correct-letters="correctLetters" />
  </div>

  <!-- Container for final message -->
  <GamePopup ref="popup" :word="word" @restart="restart" />
  <!-- Notification -->
  <GameNotification ref="notification" />
</template>
