<script setup>
import {computed, reactive, ref, watch} from "vue";

const props = defineProps(["width", "height", "generateRate"])

const dev = ref(false)
const init = ref(true)
const over = ref(false)
const generateRate = ref(props.generateRate || 0.1)
const width = ref(props.width || 10)
const height = ref(props.height || 10)

const stateMatrix = reactive(
    Array.from(
        {length: height.value},
        (v, y) => Array.from(
            {length: width.value}, (v, x) => {
              return {
                x,
                y,
                revealed: false,
                mine: false,
                adjacentMineCount: 0,
                flag: false
              }
            }
        )
    )
);

function generateMine(init) {
  stateMatrix.forEach((row) => {
    row.forEach((state) => {
      if (state.x !== init.x && state.y !== init.y && Math.random() < generateRate.value) {
        state.mine = true;
      }
    })
  })
  calculateAdjacentMine()
}

function revealMine(state) {
  state.revealed = true
  if (init.value) {
    generateMine(state)
    init.value = false
  }
  if (state.mine) {
    alert("game over")
    over.value = true
    revealAll()
  }
  if (state.adjacentMineCount === 0) {
    chording(state)
  }
}

function handleRightClick(event) {
  console.log('右键被点击了', event);
  const [y, x] = event.target.id.split('-')
  console.log(y, x)
  stateMatrix[y][x].flag = !stateMatrix[y][x].flag
}

function calculateAdjacentMine() {
  const dirs = [[1, 1], [0, 1], [1, 0], [-1, 0], [0, -1], [-1, -1], [1, -1], [-1, 1]]
  stateMatrix.forEach((row) => {
    row.forEach((state) => {
      dirs.forEach(([dx, dy]) => {
        const nx = state.x + dx
        const ny = state.y + dy
        if (nx < 0 || ny < 0 || nx >= row.length || ny >= stateMatrix.length) {
        } else if (stateMatrix[ny][nx].mine) {
          state.adjacentMineCount += 1
        }
      })
    })
  })
}

function chording(state) {
  if (state.mine) {
    return
  }

  state.revealed = true
  const dirs = [[0, 1], [1, 0], [-1, 0], [0, -1]]
  dirs.forEach(([dx, dy]) => {
    const nx = state.x + dx
    const ny = state.y + dy
    if (nx < 0 || ny < 0 || nx >= stateMatrix[0].length || ny >= stateMatrix.length || stateMatrix[ny][nx].revealed) {
    } else if (stateMatrix[ny][nx].adjacentMineCount === 0) {
      console.log(ny, nx)
      chording(stateMatrix[ny][nx])
    }
  })
}

function revealAll() {
  stateMatrix.forEach((row) => {
    row.forEach((state) => {
      state.revealed = true
    })
  })
}

const solved = computed(() => {
  return stateMatrix.every((row) => row.every((state) => state.revealed || state.flag && state.mine))
})

watch(solved, (newValue) => {
  if (newValue && !over.value) {
    alert("pass")
  }
})

const numColors = [
  'text-cyan-50',
  'text-teal-100',
  'text-emerald-200',
  'text-green-300',
  'text-lime-400',
  'text-yellow-500',
  'text-amber-600',
  'text-orange-700',
  'text-red-800'
]

</script>

<template>
  <div>
    <h5 class="text-center p-2">mine sweaper</h5>

    <div v-for="row in stateMatrix" class="flex">
      <div v-for="state in row"
           :id="`${state.y}-${state.x}`"
           @click="revealMine(state)"
           @contextmenu.prevent="handleRightClick"
           class="flex h-10 w-10 items-center justify-center bg-gray-500/10  hover:bg-gray-600/30 border-2"
      >
        <template v-if="state.revealed || dev">
          <template v-if="state.mine">
            <div class="text-red-600 icon-[mdi--mine]">
            </div>
          </template>
          <template v-else>
            <div :class="[numColors[state.adjacentMineCount]]">
              {{ state.adjacentMineCount }}
            </div>
          </template>
        </template>
        <template v-else-if="state.flag">
          <div class="text-yellow-500 icon-[mdi--flag]">
          </div>
        </template>
      </div>
    </div>
  </div>

</template>

<style scoped>

</style>