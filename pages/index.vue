<template>
  <div class="pt-12">
    <div class="mx-auto w-1/2">
      <p class="text-center text-5xl font-bold">15 Puzzle</p>
      <p class="mb-2 text-center text-xl">
        Click on tiles or use arrow keys to control the board
      </p>
      <TransitionGroup
        name="grid-transition"
        tag="div"
        class="mx-auto flex w-full flex-wrap rounded-lg bg-gray-950 p-4 dark:ring-2 dark:ring-white"
      >
        <div
          v-for="tile in displayGrid"
          :key="tile"
          class="flex w-1/4 flex-col p-2"
        >
          <div
            class="flex h-36 w-36 items-center justify-center rounded-lg"
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
              class="flex text-center font-serif text-7xl font-bold text-yellow-500"
            >
              {{ tile === 0 ? '' : tile }}
            </p>
          </div>
        </div>
      </TransitionGroup>
      <div class="flex flex-row space-x-4">
        <div class="flex w-1/2 flex-col">
          <UButton class="mt-2" block @click="reset"> Reset (R) </UButton>
        </div>
        <div class="flex w-1/2 flex-col">
          <UButton class="mt-2" block @click="scramble"> Scramble (S) </UButton>
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
      <UButton
        :icon="
          isDark ? 'i-heroicons-moon-20-solid' : 'i-heroicons-sun-20-solid'
        "
        color="gray"
        class="fixed right-0 top-0 m-4"
        size="xl"
        @click="isDark = !isDark"
      />
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
        moveUp();
        break;
      case 'ArrowDown':
        e.preventDefault();
        moveDown();
        break;
      case 'ArrowLeft':
        e.preventDefault();
        moveLeft();
        break;
      case 'ArrowRight':
        e.preventDefault();
        moveRight();
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
  transition: transform 0.5s ease-in-out;
}
</style>
