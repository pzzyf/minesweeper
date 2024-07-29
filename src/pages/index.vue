<!-- eslint-disable no-alert -->
<script setup lang="ts">
import type { BlockState } from '~/types.ts'

const WIDTH = 5
const HEIGHT = 5

const numberColors = [
  'text-tan-500',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]

const state = ref(Array.from({ length: HEIGHT }, (_, y: number) => {
  return Array.from({ length: WIDTH }, (_, x: number): BlockState => {
    return {
      x,
      y,
      revealed: false,
      adjacentMines: 0,
    }
  })
}))

// continue 退出当前循环
function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1)
        continue
      if (Math.abs(initial.y - block.y) <= 1)
        continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

const directions = [
  [1, 0],
  [1, 1],
  [1, -1],
  [0, 1],
  [0, -1],
  [-1, 0],
  [-1, 1],
  [-1, -1],
]

function updateNumbers() {
  state.value.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block)
        .forEach((b) => {
          if (b.mine)
            block.adjacentMines += 1
        })
    })
  })
}

function expendZero(block: BlockState) {
  if (block.adjacentMines)
    return

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

const mineGenerated = ref(false)

function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
}

function onClick(block: BlockState) {
  if (!mineGenerated.value) {
    generateMines(block)
    mineGenerated.value = true
  }
  block.revealed = true
  if (block.mine)
    // eslint-disable-next-line no-alert
    alert('BOOM!')
  expendZero(block)
}

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'
  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'

  return block.mine
    ? 'bg-red-500/50'
    : numberColors[block.adjacentMines]
}

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    return state.value[y2][x2]
  })
    .filter(Boolean) as BlockState[]
}

watch([() => state.value, mineGenerated.value], checkGameState, { deep: true, immediate: true })

function checkGameState() {
  if (!mineGenerated.value)
    return
  const blocks = state.value.flat()

  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You cheat!')
    else
      alert('You win!')
  }
}
</script>

<template>
  <div>
    <div class="mb-5">
      minisweeper
    </div>
    <div v-for="row, y in state" :key="y" class="flex items-center justify-center">
      <button
        v-for="block, x in row" :key="x" class="h-10 w-10 flex items-center justify-center border"
        :class="getBlockClass(block)"
        @click="onClick(block)"
        @contextmenu.prevent="onRightClick(block)"
      >
        <template v-if="block.flagged">
          <div i-mdi-flag text-red>
            🚩
          </div>
        </template>
        <template v-else-if="block.revealed">
          <div v-if="block.mine" i-mdi-mine>
            💣
          </div>
          <div v-else>
            {{ block.adjacentMines }}
          </div>
        </template>
      </button>
    </div>
    <Footer />
  </div>
</template>
