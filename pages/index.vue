<template>
  <div class="pt-12">
    <div class="mx-auto w-full px-2 lg:w-1/2 lg:px-0">
      <p class="text-center text-5xl font-bold">15 Puzzle</p>
      <p class="mb-2 text-center text-xl">
        Click on tiles or use arrow keys to control the board
      </p>
      <TransitionGroup
        ref="board"
        name="grid-transition"
        tag="div"
        class="mx-auto flex w-full flex-wrap rounded-lg bg-gray-950 p-4 dark:ring-2 dark:ring-white"
      >
        <div
          v-for="tile in displayGrid"
          :key="tile"
          class="flex aspect-square w-1/4 flex-col lg:p-2"
        >
          <div
            class="flex h-full w-full items-center justify-center lg:rounded-lg"
            :class="
              tile === 0
                ? 'invisible'
                : redSquares.includes(tile)
                ? 'bg-red-600'
                : 'bg-white'
            "
            @click="move(tile)"
          >
            <p
              class="flex text-center font-serif text-5xl font-bold text-yellow-500 lg:text-7xl"
            >
              {{ tile === 0 ? '' : tile }}
            </p>
          </div>
        </div>
      </TransitionGroup>
      <div class="mt-2 flex flex-row space-x-4">
        <div class="flex w-1/2 flex-col">
          <button
            class="rounded-lg bg-blue-500 py-1 text-white hover:bg-blue-600"
            @click="reset"
          >
            Reset (R)
          </button>
        </div>
        <div class="flex w-1/2 flex-col">
          <button
            class="rounded-lg bg-blue-500 py-1 text-white hover:bg-blue-600"
            @click="scramble"
          >
            Scramble (S)
          </button>
        </div>
      </div>
      <p
        class="mt-2 cursor-pointer text-center text-xl"
        @click="showMoves = !showMoves"
      >
        Moves: {{ moves }}
      </p>
      <div v-if="showMoves">
        <p v-for="move in movesList" :key="move" class="text-center">
          {{ move }}
        </p>
      </div>
    </div>
    <ClientOnly>
      <button class="absolute right-0 top-0 m-4" @click="isDark = !isDark">
        Toggle Theme
      </button>
      <template #fallback>
        <div class="h-8 w-8" />
      </template>
    </ClientOnly>
  </div>
</template>

<script setup lang="ts">
const grid = ref([
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9, 10, 11, 12],
  [13, 14, 15, 0],
]);

const displayGrid = computed(() => {
  const tiles = [];
  for (const row of grid.value) {
    for (const tile of row) {
      tiles.push(tile);
    }
  }
  return tiles;
});

const board = ref(null);
useSwipe(board, {
  onSwipeEnd(e: TouchEvent, direction) {
    e.preventDefault();
    switch (direction) {
      case 'up':
        moveDown();
        break;
      case 'down':
        moveUp();
        break;
      case 'left':
        moveRight();
        break;
      case 'right':
        moveLeft();
        break;
    }
  },
});

const redSquares = [2, 6, 10, 14, 4, 8, 12];
const zeroLocation = [3, 3];
const moves = ref(0);
const movesList = ref([]);
const showMoves = ref(false);

function reset() {
  grid.value = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
    [13, 14, 15, 0],
  ];
  zeroLocation[0] = 3;
  zeroLocation[1] = 3;
  moves.value = 0;
  movesList.value = [];
}

function scramble() {
  reset();
  for (let i = 0; i < 250; i++) {
    const random = Math.floor(Math.random() * 4);
    switch (random) {
      case 0:
        moveUp();
        break;
      case 1:
        moveDown();
        break;
      case 2:
        moveLeft();
        break;
      case 3:
        moveRight();
        break;
    }
  }
  for (let i = 0; i < 3; i++) {
    moveDown();
    moveRight();
  }
  moves.value = 0;
  movesList.value = [];
}

function move(tile: number) {
  const [row, col] = zeroLocation;
  const tileLocation = [0, 0];
  for (let i = 0; i < grid.value.length; i++) {
    for (let j = 0; j < grid.value[i].length; j++) {
      if (grid.value[i][j] === tile) {
        tileLocation[0] = i;
        tileLocation[1] = j;
        break;
      }
    }
  }
  if (row === tileLocation[0]) {
    if (col < tileLocation[1]) {
      for (let i = col; i < tileLocation[1]; i++) {
        moveRight();
      }
    } else {
      for (let i = col; i > tileLocation[1]; i--) {
        moveLeft();
      }
    }
  } else if (col === tileLocation[1]) {
    if (row < tileLocation[0]) {
      for (let i = row; i < tileLocation[0]; i++) {
        moveDown();
      }
    } else {
      for (let i = row; i > tileLocation[0]; i--) {
        moveUp();
      }
    }
  }
}

function moveUp() {
  if (zeroLocation[0] === 0) return;
  const temp = grid.value[zeroLocation[0] - 1][zeroLocation[1]];
  grid.value[zeroLocation[0] - 1][zeroLocation[1]] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[0] -= 1;
  moves.value += 1;
  movesList.value.unshift('up');
}

function moveDown() {
  if (zeroLocation[0] === 3) return;
  const temp = grid.value[zeroLocation[0] + 1][zeroLocation[1]];
  grid.value[zeroLocation[0] + 1][zeroLocation[1]] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[0] += 1;
  moves.value += 1;
  movesList.value.unshift('down');
}

function moveLeft() {
  if (zeroLocation[1] === 0) return;
  const temp = grid.value[zeroLocation[0]][zeroLocation[1] - 1];
  grid.value[zeroLocation[0]][zeroLocation[1] - 1] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[1] -= 1;
  moves.value += 1;
  movesList.value.unshift('left');
}

function moveRight() {
  if (zeroLocation[1] === 3) return;
  const temp = grid.value[zeroLocation[0]][zeroLocation[1] + 1];
  grid.value[zeroLocation[0]][zeroLocation[1] + 1] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[1] += 1;
  moves.value += 1;
  movesList.value.unshift('right');
}

onKeyStroke(
  ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'r', 's'],
  (e) => {
    switch (e.key) {
      case 'ArrowUp':
        e.preventDefault();
        moveDown();
        break;
      case 'ArrowDown':
        e.preventDefault();
        moveUp();
        break;
      case 'ArrowLeft':
        e.preventDefault();
        moveRight();
        break;
      case 'ArrowRight':
        e.preventDefault();
        moveLeft();
        break;
      case 'r':
        reset();
        break;
      case 's':
        scramble();
        break;
    }
  },
);

// dark mode controls
const colorMode = useColorMode();
const isDark = computed({
  get() {
    return colorMode.value === 'dark';
  },
  set() {
    colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark';
  },
});
</script>

<style scoped>
.grid-transition-move {
  transition: transform 0.25s ease-in-out;
}
</style>
