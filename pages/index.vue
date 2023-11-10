<template>
  <div class="pt-12">
    <div class="mx-auto w-fit">
      <p class="text-center text-5xl font-bold">15 Puzzle</p>
      <p class="mb-2 text-center text-xl">
        Use arrow keys to control the board. Press 'R' to reset.
      </p>
      <div
        class="flex flex-col space-y-2 rounded-lg bg-gray-950 p-4 dark:ring-2 dark:ring-white"
      >
        <div v-for="i in 4" :key="i" class="flex flex-row space-x-2">
          <div v-for="j in 4" :key="j" class="flex flex-col">
            <div
              class="flex h-36 w-36 items-center justify-center rounded-lg"
              :class="
                grid[i - 1][j - 1] === 0
                  ? 'bg-gray-950'
                  : redSquares.includes(grid[i - 1][j - 1])
                  ? 'bg-red-600'
                  : 'bg-white'
              "
            >
              <p
                class="flex text-center font-serif text-7xl font-bold text-yellow-500"
              >
                {{ grid[i - 1][j - 1] === 0 ? '' : grid[i - 1][j - 1] }}
              </p>
            </div>
          </div>
        </div>
      </div>
      <UButton class="mt-2" block @click="reset"> Reset </UButton>
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

onKeyStroke('r', (e) => {
  reset();
});

onKeyStroke('ArrowUp', (e) => {
  e.preventDefault();
  if (zeroLocation[0] === 0) return;
  const temp = grid.value[zeroLocation[0] - 1][zeroLocation[1]];
  grid.value[zeroLocation[0] - 1][zeroLocation[1]] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[0] -= 1;
  moves.value += 1;
  movesList.value.unshift('up');
});

onKeyStroke('ArrowDown', (e) => {
  e.preventDefault();
  if (zeroLocation[0] === 3) return;
  const temp = grid.value[zeroLocation[0] + 1][zeroLocation[1]];
  grid.value[zeroLocation[0] + 1][zeroLocation[1]] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[0] += 1;
  moves.value += 1;
  movesList.value.unshift('down');
});

onKeyStroke('ArrowLeft', (e) => {
  e.preventDefault();
  if (zeroLocation[1] === 0) return;
  const temp = grid.value[zeroLocation[0]][zeroLocation[1] - 1];
  grid.value[zeroLocation[0]][zeroLocation[1] - 1] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[1] -= 1;
  moves.value += 1;
  movesList.value.unshift('left');
});

onKeyStroke('ArrowRight', (e) => {
  e.preventDefault();
  if (zeroLocation[1] === 3) return;
  const temp = grid.value[zeroLocation[0]][zeroLocation[1] + 1];
  grid.value[zeroLocation[0]][zeroLocation[1] + 1] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[1] += 1;
  moves.value += 1;
  movesList.value.unshift('right');
});

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
