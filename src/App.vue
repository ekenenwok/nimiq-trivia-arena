<script setup>
import { ref, onMounted, computed } from 'vue'

const nimiq = ref(null)
const walletAddress = ref('')
const gameState = ref('home')
const currentQuestion = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const timeLeft = ref(15)
const timer = ref(null)
const nimEarned = ref(0)
const isConnecting = ref(true)
const showCorrect = ref(false)
const showWrong = ref(false)
const difficulty = ref('medium')
const streak = ref(0)
const bestStreak = ref(0)
const totalGamesPlayed = ref(0)
const bestScore = ref(0)
const dailyBonusClaimed = ref(false)
const showDailyBonus = ref(false)
const leaderboard = ref([])
const showLeaderboard = ref(false)
const playerName = ref('')
const nameEntered = ref(false)
const shuffledQuestions = ref([])
const audioCtx = ref(null)
const soundEnabled = ref(true)

const allQuestions = {
  easy: [
    { question: "What is the native currency of Nimiq?", options: ["ETH", "BTC", "NIM", "SOL"], answer: 2, emoji: "💎" },
    { question: "What does Web3 refer to?", options: ["A website version", "Decentralized internet", "A browser", "Google product"], answer: 1, emoji: "🌐" },
    { question: "What is a crypto wallet?", options: ["Physical wallet", "Key storage", "Mining tool", "Exchange"], answer: 1, emoji: "👛" },
    { question: "What is Bitcoin?", options: ["A company", "Digital currency", "A bank", "Social media"], answer: 1, emoji: "₿" },
    { question: "What is blockchain?", options: ["A chain store", "Distributed ledger", "A bank system", "Cloud storage"], answer: 1, emoji: "⛓️" },
    { question: "What is an NFT?", options: ["New Financial Token", "Non-Fungible Token", "Network Fee", "Nimiq Fast Transfer"], answer: 1, emoji: "🎨" },
    { question: "What is DeFi?", options: ["Digital Finance", "Decentralized Finance", "Direct Finance", "Default Finance"], answer: 1, emoji: "🏦" },
    { question: "What is mining in crypto?", options: ["Gold digging", "Validating transactions", "Hacking", "Trading"], answer: 1, emoji: "⛏️" },
  ],
  medium: [
    { question: "What consensus does Nimiq use?", options: ["Proof of Work", "Proof of Stake", "Albatross PoS", "Delegated PoS"], answer: 2, emoji: "🦅" },
    { question: "What is gas in Ethereum?", options: ["Physical gas", "Transaction fee", "Mining reward", "Token type"], answer: 1, emoji: "⛽" },
    { question: "What is a smart contract?", options: ["Legal document", "Self-executing code", "Crypto exchange", "Digital signature"], answer: 1, emoji: "📜" },
    { question: "What is a private key?", options: ["Hotel key", "Secret crypto key", "Public password", "API key"], answer: 1, emoji: "🔑" },
    { question: "What is staking?", options: ["Eating steak", "Locking crypto for rewards", "Selling coins", "Mining"], answer: 1, emoji: "🥩" },
    { question: "What is a DAO?", options: ["A person", "Decentralized Autonomous Org", "Digital Asset Order", "Data Access Object"], answer: 1, emoji: "🏛️" },
    { question: "What is USDT?", options: ["US Digital Token", "Tether stablecoin", "US Treasury", "Unified Standard Token"], answer: 1, emoji: "💵" },
    { question: "What is MetaMask?", options: ["A face mask", "Ethereum wallet", "A crypto exchange", "A blockchain"], answer: 1, emoji: "🦊" },
  ],
  hard: [
    { question: "What is Nimiq's block time target?", options: ["10 minutes", "1 minute", "60 seconds", "30 seconds"], answer: 1, emoji: "⏱️" },
    { question: "What hashing algorithm does Bitcoin use?", options: ["SHA-256", "SHA-512", "MD5", "Scrypt"], answer: 0, emoji: "🔐" },
    { question: "What is a Merkle tree?", options: ["A real tree", "Data structure for verification", "Mining algorithm", "Consensus method"], answer: 1, emoji: "🌳" },
    { question: "What is the Lightning Network?", options: ["Weather tech", "Bitcoin Layer 2", "Ethereum upgrade", "A crypto exchange"], answer: 1, emoji: "⚡" },
    { question: "What is EIP-1559?", options: ["Ethereum improvement for fees", "A Bitcoin fork", "NFT standard", "DeFi protocol"], answer: 0, emoji: "🔥" },
    { question: "What is a 51% attack?", options: ["Tax evasion", "Controlling majority hash rate", "Phishing attack", "Smart contract bug"], answer: 1, emoji: "⚔️" },
    { question: "What is zero-knowledge proof?", options: ["Knowing nothing", "Proving without revealing info", "Empty blockchain", "Anonymous mining"], answer: 1, emoji: "🕵️" },
    { question: "What is the Ethereum Virtual Machine?", options: ["A gaming PC", "Smart contract runtime", "Mining software", "Crypto wallet"], answer: 1, emoji: "💻" },
  ]
}

const difficultyConfig = {
  easy: { time: 20, nimPerAnswer: 3, label: '😊 Easy', color: '#4ade80' },
  medium: { time: 15, nimPerAnswer: 5, label: '🔥 Medium', color: '#fbbf24' },
  hard: { time: 10, nimPerAnswer: 10, label: '💀 Hard', color: '#ef4444' }
}

const config = computed(() => difficultyConfig[difficulty.value])

function initAudio() {
  if (!audioCtx.value) {
    audioCtx.value = new (window.AudioContext || window.webkitAudioContext)()
  }
}

function playSound(type) {
  if (!soundEnabled.value) return
  initAudio()
  const ctx = audioCtx.value
  if (!ctx) return

  const play = (freq, start, duration, endFreq) => {
    const osc = ctx.createOscillator()
    const gain = ctx.createGain()
    osc.connect(gain)
    gain.connect(ctx.destination)
    osc.frequency.setValueAtTime(freq, ctx.currentTime + start)
    if (endFreq) osc.frequency.exponentialRampToValueAtTime(endFreq, ctx.currentTime + start + duration)
    gain.gain.setValueAtTime(0.3, ctx.currentTime + start)
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + start + duration)
    osc.start(ctx.currentTime + start)
    osc.stop(ctx.currentTime + start + duration)
  }

  if (type === 'correct') {
    play(523, 0, 0.15)
    play(659, 0.1, 0.15)
    play(784, 0.2, 0.25)
  } else if (type === 'wrong') {
    play(300, 0, 0.1, 200)
    play(200, 0.15, 0.2)
  } else if (type === 'start') {
    play(400, 0, 0.1)
    play(500, 0.1, 0.1)
    play(700, 0.2, 0.2)
  } else if (type === 'win') {
    play(523, 0, 0.15)
    play(659, 0.15, 0.15)
    play(784, 0.3, 0.15)
    play(1047, 0.45, 0.4)
  } else if (type === 'tick') {
    play(800, 0, 0.05)
  } else if (type === 'daily') {
    play(600, 0, 0.1)
    play(800, 0.1, 0.1)
    play(1000, 0.2, 0.3)
  }
}

onMounted(async () => {
  const savedName = localStorage.getItem('playerName')
  if (savedName) { playerName.value = savedName; nameEntered.value = true }
  const savedBest = localStorage.getItem('bestScore')
  if (savedBest) bestScore.value = parseInt(savedBest)
  const savedGames = localStorage.getItem('totalGames')
  if (savedGames) totalGamesPlayed.value = parseInt(savedGames)
  const savedBoard = localStorage.getItem('leaderboard')
  if (savedBoard) leaderboard.value = JSON.parse(savedBoard)
  const lastBonus = localStorage.getItem('lastDailyBonus')
  const today = new Date().toDateString()
  if (lastBonus !== today) showDailyBonus.value = true
  else dailyBonusClaimed.value = true

  try {
    if (window.nimiq) {
      nimiq.value = window.nimiq
      const accounts = await nimiq.value.listAccounts()
      if (accounts.length > 0) walletAddress.value = accounts[0]
    }
  } catch(e) {}
  isConnecting.value = false
})

function claimDailyBonus() {
  nimEarned.value += 10
  dailyBonusClaimed.value = true
  showDailyBonus.value = false
  localStorage.setItem('lastDailyBonus', new Date().toDateString())
  playSound('daily')
}

function saveName() {
  if (playerName.value.trim().length < 2) return
  localStorage.setItem('playerName', playerName.value.trim())
  nameEntered.value = true
  playSound('start')
}

function startGame() {
  const pool = allQuestions[difficulty.value]
  shuffledQuestions.value = [...pool].sort(() => Math.random() - 0.5).slice(0, 5)
  currentQuestion.value = 0
  score.value = 0
  nimEarned.value = 0
  selectedAnswer.value = null
  showCorrect.value = false
  showWrong.value = false
  streak.value = 0
  gameState.value = 'playing'
  playSound('start')
  startTimer()
}

function startTimer() {
  timeLeft.value = config.value.time
  clearInterval(timer.value)
  timer.value = setInterval(() => {
    timeLeft.value--
    if (timeLeft.value <= 3) playSound('tick')
    if (timeLeft.value <= 0) {
      clearInterval(timer.value)
      streak.value = 0
      playSound('wrong')
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
    streak.value++
    if (streak.value > bestStreak.value) bestStreak.value = streak.value
    const bonus = streak.value >= 3 ? config.value.nimPerAnswer * 2 : config.value.nimPerAnswer
    nimEarned.value += bonus
    showCorrect.value = true
    showWrong.value = false
    playSound('correct')
  } else {
    streak.value = 0
    showWrong.value = true
    showCorrect.value = false
    playSound('wrong')
  }
  setTimeout(() => {
    showCorrect.value = false
    showWrong.value = false
    nextQuestion()
  }, 1200)
}

function nextQuestion() {
  selectedAnswer.value = null
  if (currentQuestion.value < shuffledQuestions.value.length - 1) {
    currentQuestion.value++
    startTimer()
  } else {
    endGame()
  }
}

function endGame() {
  totalGamesPlayed.value++
  localStorage.setItem('totalGames', totalGamesPlayed.value)
  if (score.value > bestScore.value) {
    bestScore.value = score.value
    localStorage.setItem('bestScore', bestScore.value)
  }
  if (nameEntered.value) {
    const entry = {
      name: playerName.value,
      score: score.value,
      nim: nimEarned.value,
      difficulty: difficulty.value,
      date: new Date().toLocaleDateString()
    }
    leaderboard.value.unshift(entry)
    leaderboard.value = leaderboard.value.slice(0, 10)
    localStorage.setItem('leaderboard', JSON.stringify(leaderboard.value))
  }
  if (score.value >= 4) playSound('win')
  else playSound('wrong')
  gameState.value = 'result'
}

function getAnswerClass(index) {
  if (selectedAnswer.value === null) return 'answer-btn'
  if (index === shuffledQuestions.value[currentQuestion.value].answer) return 'answer-btn correct'
  if (index === selectedAnswer.value) return 'answer-btn wrong'
  return 'answer-btn dimmed'
}

function shortAddress(addr) {
  if (!addr) return 'Demo Mode'
  return addr.slice(0, 8) + '...' + addr.slice(-6)
}

async function claimReward() {
  if (nimiq.value && nimEarned.value > 0 && walletAddress.value) {
    try {
      await nimiq.value.sendTransaction({
        to: walletAddress.value,
        amount: nimEarned.value * 100000,
        message: 'Nimiq Trivia Arena reward!'
      })
    } catch(e) {}
  }
  gameState.value = 'home'
}
</script>

<template>
  <div class="app">

    <!-- LOADING -->
    <div v-if="isConnecting" class="screen center">
      <div class="logo-pulse">🧠</div>
      <p class="connecting-text">Loading Nimiq Trivia Arena...</p>
      <div class="spinner"></div>
    </div>

    <!-- NAME ENTRY -->
    <div v-else-if="!nameEntered" class="screen center">
      <div class="logo-pulse">🧠</div>
      <h2 class="title">Welcome!</h2>
      <p class="subtitle">Enter your name for the leaderboard</p>
      <input v-model="playerName" placeholder="Your name..." class="name-input" maxlength="12" @keyup.enter="saveName" />
      <button class="play-btn" @click="saveName">Let's Play! 🎮</button>
    </div>

    <!-- HOME -->
    <div v-else-if="gameState === 'home'" class="screen">

      <!-- Daily Bonus -->
      <div v-if="showDailyBonus" class="bonus-overlay">
        <div class="bonus-card">
          <div class="bonus-emoji">🎁</div>
          <h3 class="bonus-title">Daily Bonus!</h3>
          <p class="bonus-sub">Come back every day for free NIM!</p>
          <p class="bonus-amount">+10 NIM</p>
          <button class="play-btn" @click="claimDailyBonus">Claim Now! 🎉</button>
        </div>
      </div>

      <div class="home-header">
        <div class="wallet-badge">
          <span class="dot"></span>
          {{ shortAddress(walletAddress) }}
        </div>
        <div class="header-right">
          <button class="icon-btn" @click="soundEnabled = !soundEnabled">{{ soundEnabled ? '🔊' : '🔇' }}</button>
          <button class="icon-btn" @click="showLeaderboard = !showLeaderboard">🏆</button>
        </div>
      </div>

      <!-- Leaderboard -->
      <div v-if="showLeaderboard" class="leaderboard">
        <h3 class="lb-title">🏆 Leaderboard</h3>
        <div v-if="leaderboard.length === 0" class="lb-empty">No scores yet. Be the first!</div>
        <div v-for="(entry, i) in leaderboard" :key="i" class="lb-entry">
          <span class="lb-rank">{{ i === 0 ? '🥇' : i === 1 ? '🥈' : i === 2 ? '🥉' : '#' + (i+1) }}</span>
          <span class="lb-name">{{ entry.name }}</span>
          <span class="lb-diff">{{ entry.difficulty }}</span>
          <span class="lb-score">{{ entry.score }}/5</span>
          <span class="lb-nim">{{ entry.nim }} NIM</span>
        </div>
      </div>

      <div class="hero">
        <div class="logo-pulse">🧠</div>
        <h1 class="title">Nimiq Trivia<br><span class="gold">Arena</span></h1>
        <p class="subtitle">Answer questions • Earn NIM • Win big</p>
      </div>

      <div class="player-stats">
        <div class="ps-item">
          <span class="ps-label">Player</span>
          <span class="ps-value">{{ playerName }}</span>
        </div>
        <div class="ps-item">
          <span class="ps-label">Best Score</span>
          <span class="ps-value gold">{{ bestScore }}/5</span>
        </div>
        <div class="ps-item">
          <span class="ps-label">Games Played</span>
          <span class="ps-value">{{ totalGamesPlayed }}</span>
        </div>
      </div>

      <div class="diff-section">
        <p class="diff-label">Select Difficulty</p>
        <div class="diff-btns">
          <button v-for="d in ['easy', 'medium', 'hard']" :key="d" @click="difficulty = d" :class="['diff-btn', difficulty === d ? 'active-' + d : '']">
            {{ difficultyConfig[d].label }}
          </button>
        </div>
        <p class="diff-info">{{ config.time }}s per question • {{ config.nimPerAnswer }} NIM per correct answer{{ streak >= 3 ? ' • 2x streak bonus!' : '' }}</p>
      </div>

      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-icon">💰</div>
          <div class="stat-label">Prize Pool</div>
          <div class="stat-value">25 NIM</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">🔥</div>
          <div class="stat-label">Streak Bonus</div>
          <div class="stat-value">2x NIM</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">🎁</div>
          <div class="stat-label">Daily Bonus</div>
          <div class="stat-value">{{ dailyBonusClaimed ? '✅ Claimed' : '10 NIM' }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">💀</div>
          <div class="stat-label">Hard Bonus</div>
          <div class="stat-value">10 NIM/Q</div>
        </div>
      </div>

      <button class="play-btn" @click="startGame">🎮 Start Playing</button>
      <p class="footer-note">Powered by Nimiq Pay ⚡</p>
    </div>

    <!-- PLAYING -->
    <div v-else-if="gameState === 'playing'" class="screen">
      <div class="game-header">
        <div class="progress-info">
          <span class="q-count">Q{{ currentQuestion + 1 }}/{{ shuffledQuestions.length }}</span>
          <span class="streak-badge" v-if="streak >= 2">🔥 x{{ streak }} Streak!</span>
          <span class="score-live">⭐ {{ score }}</span>
        </div>
        <div class="timer-bar">
          <div class="timer-fill" :style="{ width: (timeLeft / config.time * 100) + '%' }" :class="timeLeft <= 3 ? 'danger' : ''"></div>
        </div>
        <div class="timer-row">
          <span class="timer-text" :class="timeLeft <= 3 ? 'danger-text' : ''">⏱️ {{ timeLeft }}s</span>
          <span class="nim-live">💎 {{ nimEarned }} NIM</span>
        </div>
      </div>

      <div class="question-card" v-if="shuffledQuestions.length">
        <div class="diff-tag" :style="{ background: config.color }">{{ config.label }}</div>
        <div class="q-emoji">{{ shuffledQuestions[currentQuestion].emoji }}</div>
        <p class="question-text">{{ shuffledQuestions[currentQuestion].question }}</p>
      </div>

      <div class="feedback correct-flash" v-if="showCorrect">✅ Correct! +{{ streak >= 3 ? config.nimPerAnswer * 2 : config.nimPerAnswer }} NIM {{ streak >= 3 ? '🔥 STREAK BONUS!' : '' }}</div>
      <div class="feedback wrong-flash" v-if="showWrong">❌ Wrong! Keep going!</div>

      <div class="answers" v-if="shuffledQuestions.length">
        <button v-for="(option, i) in shuffledQuestions[currentQuestion].options" :key="i" @click="selectAnswer(i)" :class="getAnswerClass(i)">
          <span class="answer-letter">{{ ['A','B','C','D'][i] }}</span>
          {{ option }}
        </button>
      </div>
    </div>

    <!-- RESULT -->
    <div v-else class="screen center">
      <div class="result-emoji">{{ score >= 4 ? '🏆' : score >= 2 ? '😊' : '😅' }}</div>
      <h2 class="result-title">{{ score >= 4 ? 'CHAMPION!' : score >= 2 ? 'Good Job!' : 'Try Again!' }}</h2>
      <p class="result-sub">{{ playerName }} scored {{ score }}/{{ shuffledQuestions.length }}</p>

      <div class="reward-card">
        <p class="reward-label">Total NIM Earned</p>
        <p class="reward-amount">{{ nimEarned }} NIM</p>
        <p class="reward-sub">Best Streak: 🔥 {{ bestStreak }}</p>
        <p class="reward-sub" v-if="score >= bestScore && score > 0">🎉 New Personal Best!</p>
      </div>

      <div class="score-breakdown">
        <div class="breakdown-item" v-for="n in shuffledQuestions.length" :key="n">
          <span>Q{{ n }}</span>
          <span>{{ n <= score ? '✅' : '❌' }}</span>
        </div>
      </div>

      <button class="play-btn" @click="claimReward">
        {{ nimEarned > 0 && walletAddress ? '💰 Claim & Play Again' : '🔄 Play Again' }}
      </button>
      <button class="home-btn" @click="gameState = 'home'">🏠 Home</button>
    </div>

  </div>
</template>

<style scoped>
* { box-sizing: border-box; margin: 0; padding: 0; }

.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f0225 0%, #0d1b4b 50%, #0a1628 100%);
  color: white;
  font-family: 'Segoe UI', sans-serif;
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
  gap: 16px;
}

.logo-pulse { font-size: 72px; animation: pulse-scale 2s ease-in-out infinite; }

@keyframes pulse-scale {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

.connecting-text { color: rgba(255,255,255,0.7); font-size: 16px; }

.spinner {
  width: 40px; height: 40px;
  border: 3px solid rgba(255,255,255,0.1);
  border-top-color: #fbbf24;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

.name-input {
  width: 100%;
  background: rgba(255,255,255,0.1);
  border: 2px solid rgba(255,255,255,0.2);
  color: white;
  padding: 14px 18px;
  border-radius: 14px;
  font-size: 16px;
  text-align: center;
  outline: none;
}

.name-input:focus { border-color: #fbbf24; }
.name-input::placeholder { color: rgba(255,255,255,0.3); }

.bonus-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 20px;
}

.bonus-card {
  background: linear-gradient(135deg, #1a0533, #0d1b4b);
  border: 2px solid #fbbf24;
  border-radius: 24px;
  padding: 32px 24px;
  text-align: center;
  max-width: 300px;
  width: 10
