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
const audioContext = ref(null)

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

onMounted(async () => {
  const savedName = localStorage.getItem('playerName')
  if (savedName) { playerName.value = savedName; nameEntered.value = true }
  
  const savedBest = localStorage.getItem('bestScore')
  if (savedBest) bestScore.value = parseInt(savedBest)
  
  const savedGames = localStorage.getItem('totalGames')
  if (savedGames) totalGamesPlayed.value = parseInt
