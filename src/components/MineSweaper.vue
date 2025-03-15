<script setup>
import {computed, reactive, ref, watch} from "vue";

const props = defineProps(["width", "height"])

const dev = ref(false)
const init = ref(true)
const over = ref(false)

const WIDTH = props.width || 10
const HEIGHT = props.height || 10

const stateMatrix = reactive(
    Array.from(
        {length: HEIGHT},
        (v, y) => Array.from(
            {length: WIDTH}, (v, x) => {
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
      if (state.x !== init.x && state.y !== init.y && Math.random() < 0.3) {
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
  if (newValue&&!over.value) {
    alert("pass")
  }
})

const numColors = [
  'text-white-300',
  'text-amber-300',
  'text-green-300',
  'text-red-300',
  'text-lime-300',
  'text-yellow-300',
  'text-emerald-300',
  'text-cyan-300',
  'text-fuchsia-300'
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
           class="flex h-2 w-2 p-4 items-center justify-center bg-gray-300/10  hover:bg-gray-600 border-1"
      >
        <template v-if="state.revealed || dev">
          <template v-if="state.mine">
            <div class="text-red-600">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                <path fill="currentColor"
                      d="M23 13v-2h-3.07a8 8 0 0 0-1.62-3.9l2.19-2.17l-1.43-1.43l-2.17 2.19A8 8 0 0 0 13 4.07V1h-2v3.07c-1.42.18-2.77.74-3.9 1.62L4.93 3.5L3.5 4.93L5.69 7.1A8 8 0 0 0 4.07 11H1v2h3.07c.18 1.42.74 2.77 1.62 3.9L3.5 19.07l1.43 1.43l2.17-2.19c1.13.88 2.48 1.44 3.9 1.62V23h2v-3.07c1.42-.18 2.77-.74 3.9-1.62l2.17 2.19l1.43-1.43l-2.19-2.17a8 8 0 0 0 1.62-3.9zM12 8a4 4 0 0 0-4 4H6a6 6 0 0 1 6-6z"/>
              </svg>
            </div>
          </template>
          <template v-else>
            <div :class="[numColors[state.adjacentMineCount]]">
              {{ state.adjacentMineCount }}
            </div>
          </template>
        </template>
        <template v-else-if="state.flag">
<!--          <div class="text-yellow-300">-->
<!--            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">-->
<!--              <path fill="currentColor" d="M7 2h2v20H7zm12 7l-8 5.6V3.4z"/>-->
<!--            </svg>-->
<!--          </div>-->
        </template>
      </div>
    </div>
  </div>

</template>

<style scoped>

</style>