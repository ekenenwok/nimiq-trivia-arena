<script setup>
import { ref } from 'vue'

const currentQuestion = ref(0)
const score = ref(0)
const gameState = ref('home')
const selectedAnswer = ref(null)
const timeLeft = ref(15)
const timer = ref(null)

const questions = [
  { question: "What is the native currency of Nimiq?", options: ["ETH", "BTC", "NIM", "SOL"], answer: 2 },
  { question: "What does Web3 refer to?", options: ["A website version", "Decentralized internet", "A browser plugin", "Google product"], answer: 1 },
  { question: "What is a crypto wallet used for?", options: ["Storing cash", "Storing private keys", "Mining coins", "Paying taxes"], answer: 1 },
  { question: "What is a smart contract?", options: ["A legal document", "Self-executing blockchain code", "A crypto exchange", "A digital signature"], answer: 1 },
  { question: "What blockchain is Nimiq built on?", options: ["Ethereum", "Nimiq own chain", "Solana", "Bitcoin"], answer: 1 }
]

function startGame() {
  currentQuestion.value = 0
  score.value = 0
  selectedAnswer.value = null
  gameState.value = 'playing'
  startTimer()
}

function startTimer() {
  timeLeft.value = 15
  clearInterval(timer.value)
  timer.value = setInterval(() => {
    timeLeft.value--
    if (timeLeft.value <= 0) {
      clearInterval(timer.value)
      nextQuestion()
    }
  }, 1000)
}

function selectAnswer(index) {
  if (selectedAnswer.value !== null) return
  selectedAnswer.value = index
  clearInterval(timer.value)
  if (index === questions[currentQuestion.value].answer) score.value++
  setTimeout(() => nextQuestion(), 1200)
}

function nextQuestion() {
  selectedAnswer.value = null
  if (currentQuestion.value < questions.length - 1) {
    currentQuestion.value++
    startTimer()
  } else {
    gameState.value = 'result'
  }
}

function getAnswerClass(index) {
  if (selectedAnswer.value === null) return 'bg-white/10 hover:bg-white/20'
  if (index === questions[currentQuestion.value].answer) return 'bg-green-500'
  if (index === selectedAnswer.value) return 'bg-red-500'
  return 'bg-white/10 opacity-50'
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-purple-900 to-blue-900 text-white p-6 font-sans">
    <div class="max-w-md mx-auto">

      <div v-if="gameState === 'home'" class="text-center space-y-6 pt-12">
        <h1 class="text-4xl font-bold">🧠 Nimiq Trivia Arena</h1>
        <p class="text-purple-200">Answer questions • Earn NIM • Win big</p>
        <div class="bg-white/10 rounded-3xl p-6 space-y-3">
          <div class="flex justify-between"><span>💰 Prize Pool</span><span class="font-bold">25 NIM</span></div>
          <div class="flex justify-between"><span>❓ Questions</span><span class="font-bold">5</span></div>
          <div class="flex justify-between"><span>⏱️ Time per Q</span><span class="font-bold">15 sec</span></div>
        </div>
        <button @click="startGame" class="w-full bg-yellow-400 hover:bg-yellow-300 transition text-black font-bold py-4 rounded-2xl text-xl">🎮 Start Playing</button>
      </div>

      <div v-else-if="gameState === 'playing'" class="space-y-6 pt-6">
        <div class="flex justify-between items-center">
          <span class="text-sm opacity-75">Question {{ currentQuestion + 1 }}/{{ questions.length }}</span>
          <span class="text-lg">⏱️ {{ timeLeft }}s</span>
        </div>
        <div class="w-full bg-white/20 rounded-full h-2">
          <div class="bg-yellow-400 h-2 rounded-full" :style="{ width: (timeLeft / 15 * 100) + '%' }"></div>
        </div>
        <div class="bg-white/10 rounded-3xl p-6">
          <p class="text-lg font-semibold">{{ questions[currentQuestion].question }}</p>
        </div>
        <div class="space-y-3">
          <button v-for="(option, i) in questions[currentQuestion].options" :key="i" @click="selectAnswer(i)" :class="getAnswerClass(i)" class="w-full text-left px-5 py-4 rounded-2xl transition font-medium">{{ option }}</button>
        </div>
        <div class="text-center text-sm opacity-75">Score: {{ score }}</div>
      </div>

      <div v-else class="text-center space-y-6 pt-12">
        <div class="text-6xl">{{ score >= 4 ? '🏆' : score >= 2 ? '😊' : '😅' }}</div>
        <h2 class="text-3xl font-bold">Game Over!</h2>
        <p class="text-purple-200">You scored {{ score }} out of {{ questions.length }}</p>
        <div class="bg-white/10 rounded-3xl p-6">
          <p class="text-sm opacity-75 mb-2">NIM Earned</p>
          <p class="text-4xl font-bold text-yellow-400">{{ score * 5 }} NIM</p>
        </div>
        <button @click="startGame" class="w-full bg-yellow-400 hover:bg-yellow-300 transition text-black font-bold py-4 rounded-2xl text-xl">🔄 Play Again</button>
      </div>

    </div>
  </div>
</template>
