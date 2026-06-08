<script setup>
import { ref, onMounted } from 'vue'

const nimiq = ref(null)
const walletAddress = ref('')
const gameState = ref('home')
const currentQuestion = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const timeLeft = ref(15)
const timer = ref(null)
const betAmount = ref(100000)
const nimEarned = ref(0)
const isConnecting = ref(true)
const showCorrect = ref(false)

const questions = [
  { question: "What is the native currency of Nimiq?", options: ["ETH", "BTC", "NIM", "SOL"], answer: 2, emoji: "💎" },
  { question: "What does Web3 refer to?", options: ["A website version", "Decentralized internet", "A browser plugin", "Google product"], answer: 1, emoji: "🌐" },
  { question: "What is a crypto wallet used for?", options: ["Storing cash", "Storing private keys", "Mining coins", "Paying taxes"], answer: 1, emoji: "👛" },
  { question: "What is a smart contract?", options: ["A legal document", "Self-executing blockchain code", "A crypto exchange", "A digital signature"], answer: 1, emoji: "📜" },
  { question: "What blockchain is Nimiq built on?", options: ["Ethereum", "Its own chain", "Solana", "Bitcoin"], answer: 1, emoji: "⛓️" },
  { question: "What is DeFi short for?", options: ["Digital Finance", "Decentralized Finance", "Direct Finance", "Distributed Fees"], answer: 1, emoji: "🏦" },
  { question: "What is an NFT?", options: ["New Financial Token", "Non-Fungible Token", "Nimiq Fast Transfer", "Network Fee Token"], answer: 1, emoji: "🎨" },
  { question: "What is gas in Ethereum?", options: ["Physical gas", "Transaction fee", "Mining reward", "Token type"], answer: 1, emoji: "⛽" }
]

const shuffledQuestions = ref([])

onMounted(async () => {
  shuffledQuestions.value = [...questions].sort(() => Math.random() - 0.5).slice(0, 5)
  try {
    if (window.nimiq) {
      nimiq.value = window.nimiq
      const accounts = await nimiq.value.listAccounts()
      if (accounts.length > 0) walletAddress.value = accounts[0]
    }
  } catch (e) {
    console.log('Nimiq Pay not available, running in demo mode')
  }
  isConnecting.value = false
})

function startGame() {
  currentQuestion.value = 0
  score.value = 0
  nimEarned.value = 0
  selectedAnswer.value = null
  showCorrect.value = false
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
  if (index === shuffledQuestions.value[currentQuestion.value].answer) {
    score.value++
    nimEarned.value += 5
    showCorrect.value = true
  }
  setTimeout(() => {
    showCorrect.value = false
    nextQuestion()
  }, 1200)
}

function nextQuestion() {
  selectedAnswer.value = null
  if (currentQuestion.value < shuffledQuestions.value.length - 1) {
    currentQuestion.value++
    startTimer()
  } else {
    gameState.value = 'result'
  }
}

function getAnswerClass(index) {
  if (selectedAnswer.value === null) return 'answer-btn'
  if (index === shuffledQuestions.value[currentQuestion.value].answer) return 'answer-btn correct'
  if (index === selectedAnswer.value) return 'answer-btn wrong'
  return 'answer-btn dimmed'
}

async function claimReward() {
  if (!nimiq.value || nimEarned.value === 0) {
    startGame()
    return
  }
  try {
    await nimiq.value.sendTransaction({
      to: walletAddress.value,
      amount: nimEarned.value * 100000,
      message: 'Nimiq Trivia Arena reward!'
    })
  } catch (e) {
    console.log('Transaction cancelled')
  }
  startGame()
}

function shortAddress(addr) {
  if (!addr) return 'Demo Mode'
  return addr.slice(0, 8) + '...' + addr.slice(-6)
}
</script>

<template>
  <div class="app">

    <!-- LOADING -->
    <div v-if="isConnecting" class="screen center">
      <div class="logo-big">🧠</div>
      <p class="connecting">Connecting to Nimiq Pay...</p>
      <div class="spinner"></div>
    </div>

    <!-- HOME -->
    <div v-else-if="gameState === 'home'" class="screen">
      <div class="header">
        <div class="wallet-badge">
          <span class="dot"></span>
          {{ shortAddress(walletAddress) }}
        </div>
      </div>

      <div class="hero">
        <div class="logo-big">🧠</div>
        <h1 class="title">Nimiq Trivia<br><span class="gold">Arena</span></h1>
        <p class="subtitle">Answer questions • Earn NIM • Win big</p>
      </div>

      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-icon">💰</div>
          <div class="stat-label">Prize Pool</div>
          <div class="stat-value">25 NIM</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">❓</div>
          <div class="stat-label">Questions</div>
          <div class="stat-value">5</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">⏱️</div>
          <div class="stat-label">Per Question</div>
          <div class="stat-value">15s</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">🏆</div>
          <div class="stat-label">Per Answer</div>
          <div class="stat-value">5 NIM</div>
        </div>
      </div>

      <button class="play-btn" @click="startGame">
        🎮 Start Playing
      </button>

      <p class="footer-note">Powered by Nimiq Pay</p>
    </div>

    <!-- PLAYING -->
    <div v-else-if="gameState === 'playing'" class="screen">
      <div class="game-header">
        <div class="progress-info">
          <span class="q-count">{{ currentQuestion + 1 }}/{{ shuffledQuestions.length }}</span>
          <span class="score-live">⭐ {{ score }}</span>
        </div>
        <div class="timer-bar">
          <div class="timer-fill" :style="{ width: (timeLeft / 15 * 100) + '%' }" :class="timeLeft <= 5 ? 'danger' : ''"></div>
        </div>
        <div class="timer-text" :class="timeLeft <= 5 ? 'danger-text' : ''">⏱️ {{ timeLeft }}s</div>
      </div>

      <div class="question-card" v-if="shuffledQuestions.length">
        <div class="q-emoji">{{ shuffledQuestions[currentQuestion].emoji }}</div>
        <p class="question-text">{{ shuffledQuestions[currentQuestion].question }}</p>
      </div>

      <div class="correct-flash" v-if="showCorrect">✅ Correct! +5 NIM</div>

      <div class="answers" v-if="shuffledQuestions.length">
        <button
          v-for="(option, i) in shuffledQuestions[currentQuestion].options"
          :key="i"
          @click="selectAnswer(i)"
          :class="getAnswerClass(i)">
          <span class="answer-letter">{{ ['A', 'B', 'C', 'D'][i] }}</span>
          {{ option }}
        </button>
      </div>

      <div class="nim-tracker">💎 {{ nimEarned }} NIM earned so far</div>
    </div>

    <!-- RESULT -->
    <div v-else class="screen center">
      <div class="result-emoji">{{ score >= 4 ? '🏆' : score >= 2 ? '😊' : '😅' }}</div>
      <h2 class="result-title">{{ score >= 4 ? 'Amazing!' : score >= 2 ? 'Good Job!' : 'Keep Trying!' }}</h2>
      <p class="result-sub">You scored {{ score }} out of {{ shuffledQuestions.length }}</p>

      <div class="reward-card">
        <p class="reward-label">NIM Earned</p>
        <p class="reward-amount">{{ nimEarned }} NIM</p>
        <p class="reward-sub">{{ walletAddress ? 'Tap below to claim!' : 'Connect wallet to claim' }}</p>
      </div>

      <div class="result-btns">
        <button class="play-btn" @click="claimReward">
          {{ nimEarned > 0 && walletAddress ? '💰 Claim & Play Again' : '🔄 Play Again' }}
        </button>
      </div>

      <div class="score-breakdown">
        <div class="breakdown-item" v-for="n in shuffledQuestions.length" :key="n">
          <span>Q{{ n }}</span>
          <span>{{ n <= score ? '✅' : '❌' }}</span>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
* { box-sizing: border-box; margin: 0; padding: 0; }

.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a0533 0%, #0d1b4b 50%, #0a2744 100%);
  color: white;
  font-family: 'Segoe UI', sans-serif;
  overflow-x: hidden;
}

.screen {
  max-width: 420px;
  margin: 0 auto;
  padding: 20px 16px 40px;
  min-height: 100vh;
}

.screen.center {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  gap: 20px;
}

.header { margin-bottom: 24px; }

.wallet-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: rgba(255,255,255,0.1);
  border: 1px solid rgba(255,255,255,0.2);
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 13px;
  font-family: monospace;
}

.dot {
  width: 8px; height: 8px;
  background: #4ade80;
  border-radius: 50%;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.hero { text-align: center; margin-bottom: 32px; }

.logo-big { font-size: 72px; margin-bottom: 12px; }

.title {
  font-size: 36px;
  font-weight: 900;
  line-height: 1.1;
  margin-bottom: 8px;
}

.gold { color: #fbbf24; }

.subtitle { color: rgba(255,255,255,0.6); font-size: 14px; }

.stats-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 28px;
}

.stat-card {
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 16px;
  padding: 16px;
  text-align: center;
}

.stat-icon { font-size: 24px; margin-bottom: 6px; }
.stat-label { font-size: 11px; color: rgba(255,255,255,0.5); margin-bottom: 4px; }
.stat-value { font-size: 18px; font-weight: 700; color: #fbbf24; }

.play-btn {
  width: 100%;
  background: linear-gradient(135deg, #fbbf24, #f59e0b);
  color: #1a0533;
  font-weight: 800;
  font-size: 18px;
  padding: 18px;
  border-radius: 16px;
  border: none;
  cursor: pointer;
  box-shadow: 0 4px 24px rgba(251,191,36,0.4);
  transition: transform 0.1s;
}

.play-btn:active { transform: scale(0.97); }

.footer-note { text-align: center; color: rgba(255,255,255,0.3); font-size: 12px; margin-top: 16px; }

.game-header { margin-bottom: 20px; }

.progress-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.q-count { font-size: 14px; color: rgba(255,255,255,0.6); }
.score-live { font-size: 14px; font-weight: 700; color: #fbbf24; }

.timer-bar {
  width: 100%;
  height: 6px;
  background: rgba(255,255,255,0.15);
  border-radius: 3px;
  overflow: hidden;
  margin-bottom: 6px;
}

.timer-fill {
  height: 100%;
  background: linear-gradient(90deg, #4ade80, #fbbf24);
  border-radius: 3px;
  transition: width 1s linear;
}

.timer-fill.danger { background: linear-gradient(90deg, #ef4444, #dc2626); }

.timer-text { font-size: 13px; color: rgba(255,255,255,0.6); text-align: right; }
.danger-text { color: #ef4444; font-weight: 700; }

.question-card {
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 20px;
  padding: 24px;
  margin-bottom: 16px;
  text-align: center;
}

.q-emoji { font-size: 40px; margin-bottom: 12px; }

.question-text { font-size: 18px; font-weight: 600; line-height: 1.4; }

.correct-flash {
  background: rgba(74,222,128,0.2);
  border: 1px solid #4ade80;
  color: #4ade80;
  text-align: center;
  padding: 10px;
  border-radius: 12px;
  font-weight: 700;
  margin-bottom: 12px;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.answers { display: flex; flex-direction: column; gap: 10px; margin-bottom: 16px; }

.answer-btn {
  width: 100%;
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.15);
  color: white;
  font-size: 15px;
  font-weight: 500;
  padding: 16px;
  border-radius: 14px;
  cursor: pointer;
  text-align: left;
  display: flex;
  align-items: center;
  gap: 12px;
  transition: all 0.2s;
}

.answer-btn:active { transform: scale(0.98); }

.answer-btn.correct {
  background: rgba(74,222,128,0.25);
  border-color: #4ade80;
  color: #4ade80;
}

.answer-btn.wrong {
  background: rgba(239,68,68,0.25);
  border-color: #ef4444;
  color: #ef4444;
}

.answer-btn.dimmed { opacity: 0.4; }

.answer-letter {
  width: 28px; height: 28px;
  background: rgba(255,255,255,0.15);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 13px;
  flex-shrink: 0;
}

.nim-tracker {
  text-align: center;
  color: #fbbf24;
  font-weight: 600;
  font-size: 14px;
}

.result-emoji { font-size: 80px; }

.result-title { font-size: 32px; font-weight: 900; }

.result-sub { color: rgba(255,255,255,0.6); }

.reward-card {
  background: linear-gradient(135deg, rgba(251,191,36,0.15), rgba(245,158,11,0.1));
  border: 1px solid rgba(251,191,36,0.4);
  border-radius: 20px;
  padding: 24px;
  text-align: center;
  width: 100%;
}

.reward-label { font-size: 13px; color: rgba(255,255,255,0.6); margin-bottom: 8px; }
.reward-amount { font-size: 48px; font-weight: 900; color: #fbbf24; }
.reward-sub { font-size: 12px; color: rgba(255,255,255,0.5); margin-top: 6px; }

.result-btns { width: 100%; }

.score-breakdown {
  display: flex;
  gap: 12px;
  margin-top: 8px;
}

.breakdown-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  color: rgba(255,255,255,0.5);
}

.connecting { color: rgba(255,255,255,0.7); font-size: 16px; }

.spinner {
  width: 40px; height: 40px;
  border: 3px solid rgba(255,255,255,0.1);
  border-top-color: #fbbf24;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
