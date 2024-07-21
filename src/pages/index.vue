<script setup lang="ts">

interface btnProps {
  x?: number,
  y?: number,
  isMine?: boolean,
  revealed?: boolean
  adjacentMines: number
}

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]

const state = reactive(Array.from({ length: 10 }, (_, y: number) => {
  return Array.from({ length: 10 }, (_, x: number): btnProps => {
    return {
      x,
      y,
      adjacentMines: 0,
    }
  })
}))

function generateMine() {
  state.forEach((raw) => {
    raw.forEach((item) => {
      item.isMine = Math.random() < 0.3
    })
  })
}

const direction = [
  [1, 0],
  [1, 1],
  [1, -1],
  [0, 1],
  [0, -1],
  [-1, 0],
  [-1, 1],
  [-1, -1]
]

function getMineNumber() {
  state.forEach((raw, y) => {
    raw.forEach((item, x) => {
      if(item.isMine) return
      direction.forEach(([dx, dy]) => {
        const x2 = x + dx
        const y2 = y + dy
        if(x2 < 0 || x2 >= 10 || y2 <0 || y2 >= 10) return
        if(state[y2][x2].isMine){
          item.adjacentMines += 1
        }
      })

    })
  })
}

function onClick(item: btnProps) {
  item.revealed = true
  if(item.isMine){
    alert('BOOM!')
  }
}

function getBlockClass(item: btnProps) {
  if (!item.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'

  return item.isMine
    ? 'bg-red-500/50'
    : numberColors[item.adjacentMines]
}

generateMine()
getMineNumber()

</script>


<template>
  <div>
    <div class="mb-5">minisweeper</div>
    <div v-for="row in state" class="flex justify-center items-center">
      <button v-for="item in row" @click="onClick(item)"
        class="flex justify-center items-center border w-10 h-10"
        :class="getBlockClass(item)"
        >
        <template v-if="item.revealed">
          <div v-if="item.isMine" i-mdi-mine>
        </div>
        <div v-else>
          {{ item.adjacentMines }}
        </div>
        </template>
      </button>
    </div>
  </div>
</template>
