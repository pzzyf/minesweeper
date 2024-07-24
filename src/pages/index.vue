<script setup lang="ts">
const WIDTH = 5
const HEIGHT = 5

interface BlockState {
  x?: number
  y?: number
  mine?: boolean
  revealed: boolean
  adjacentMines: number
}

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

let mineGenerated = false

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  if (block.mine)
    // eslint-disable-next-line no-alert
    alert('BOOM!')
  expendZero(block)
}

function getBlockClass(item: BlockState) {
  if (!item.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'

  return item.mine
    ? 'bg-red-500/50'
    : numberColors[item.adjacentMines]
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

updateNumbers()
</script>

<template>
  <div>
    <div class="mb-5">
      minisweeper
    </div>
    <div v-for="row, y in state" :key="y" class="flex items-center justify-center">
      <button
        v-for="item, x in row" :key="x" class="h-10 w-10 flex items-center justify-center border"
        :class="getBlockClass(item)"
        @click="onClick(item)"
      >
        <template v-if="item.revealed">
          <div v-if="item.mine" i-mdi-mine />

          <div v-else>
            {{ item.adjacentMines }}
          </div>
        </template>
      </button>
    </div>
    <Footer />
  </div>
</template>
