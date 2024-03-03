<template>
  <div>
    <div class="board">
      <div
        v-for="(cell, index) in cells"
        :key="index"
        class="cell"
        :class="{ winner: winningCells.includes(index) }"
        @click="selectCell(index)"
      >
        {{ cell }}
      </div>
    </div>
    <button @click="resetGame()">Reset Game</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const cells = ref<string[]>(Array(9).fill(''))
const currentPlayer = ref('X')
const winningCells = ref<number[]>([])
const gameOver = ref(false)

const selectCell = (index: number) => {
  if (!gameOver.value && !cells.value[index]) {
    cells.value[index] = currentPlayer.value
    checkWinner()
    currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'

    console.log(currentPlayer.value)
    if (!gameOver.value) {
      // Uncomment the line below if you want to play against the AI
      computerMove()
    }
  }
}

const checkWinner = (): 'X' | 'O' | 'tie' | null => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ]

  for (let line of lines) {
    const [a, b, c] = line
    console.log(cells.value[a], cells.value[b], cells.value[c])
    if (cells.value[a] && cells.value[a] === cells.value[b] && cells.value[a] === cells.value[c]) {
      gameOver.value = true
      winningCells.value = line
      return cells.value[a] as 'X' | 'O'
    }
  }

  if (!cells.value.includes('') && currentPlayer.value === 'O') {
    gameOver.value = true
    return 'tie'
  }

  return null
}

const resetGame = () => {
  console.log('resetganme')
  cells.value = Array(9).fill('')
  currentPlayer.value = 'X'
  winningCells.value = []
  gameOver.value = false
}

const computerMove = async () => {
  let bestScore = -Infinity
  let move = null

  for (let i = 0; i < cells.value.length; i++) {
    if (cells.value[i] === '') {
      cells.value[i] = 'O'
      let score = minimax(cells.value, 0, false)
      cells.value[i] = ''
      if (score > bestScore) {
        bestScore = score
        move = i
      }
    }
  }

  if (move !== null) {
    cells.value[move] = 'O'
    checkWinner()
    currentPlayer.value = 'X'
  }
}

let i = 0
const minimax = (board: string[], depth: number, isMaximizing: boolean) => {
  if (i > 1000) return -1
  i++
  const scores = {
    X: -1,
    O: 1,
    tie: 0
  }

  const result = checkWinner()
  if (result !== null) {
    return scores[result] * (10 - depth) // Favor faster wins and slower losses
  }

  if (isMaximizing) {
    let bestScore = -Infinity
    for (let i = 0; i < board.length; i++) {
      if (board[i] === '') {
        board[i] = 'O'
        let score = minimax(board, depth + 1, false)
        board[i] = ''
        bestScore = Math.max(score, bestScore)
      }
    }
    return bestScore
  } else {
    let bestScore = Infinity
    for (let i = 0; i < board.length; i++) {
      if (board[i] === '') {
        board[i] = 'X'
        let score = minimax(board, depth + 1, true)
        board[i] = ''
        bestScore = Math.min(score, bestScore)
      }
    }
    return bestScore
  }
}
</script>

<style scoped></style>
