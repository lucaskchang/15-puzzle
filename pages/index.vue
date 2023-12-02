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
        class="mx-auto flex w-full flex-wrap bg-gray-950 p-4"
        :class="{
          'lg:rounded-lg': !classicMode,
        }"
      >
        <div
          v-for="tile in displayGrid"
          :key="tile"
          class="flex aspect-square w-1/4 flex-col"
          :class="{
            'lg:p-2': !classicMode,
          }"
        >
          <div
            class="flex h-full w-full items-center justify-center"
            :class="{
              'lg:rounded-lg': !classicMode,
              invisible: tile === 0,
              'bg-red-600': redSquares.includes(tile),
              'bg-white': !redSquares.includes(tile),
            }"
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
        <div class="flex w-1/3 flex-col">
          <button
            class="rounded-lg bg-blue-500 py-1 text-white hover:bg-blue-600"
            @click="reset"
          >
            Reset (R)
          </button>
        </div>
        <div class="flex w-1/3 flex-col">
          <button
            class="rounded-lg bg-blue-500 py-1 text-white hover:bg-blue-600"
            @click="scramble"
          >
            Scramble (S)
          </button>
        </div>
        <div class="flex w-1/3 flex-col">
          <button
            class="rounded-lg bg-blue-500 py-1 text-white hover:bg-blue-600"
            @click="solveButton"
          >
            Solve
          </button>
        </div>
      </div>
    </div>
    <div class="mt-2">
      <p class="text-center text-xl">Options</p>
      <div class="flex flex-row justify-center space-x-2">
        <p class="hidden lg:flex lg:flex-col">Classic Style</p>
        <input
          v-model="classicMode"
          type="checkbox"
          class="hidden lg:flex lg:flex-col"
        />
        <p class="flex flex-col">Show Moves</p>
        <input v-model="showMoves" type="checkbox" class="flex flex-col" />
        <p class="flex flex-col">Show Group Theory</p>
        <input
          v-model="showGroupTheory"
          type="checkbox"
          class="flex flex-col"
        />
      </div>
    </div>
    <p class="mt-2 cursor-pointer text-center text-xl">Moves: {{ moves }}</p>
    <div v-if="showGroupTheory" class="space-y-4 text-center">
      <div v-for="(permutation, i) in permutations" :key="permutation[1]">
        <p v-if="i === 0">Starting Permutation</p>
        <p v-else>{{ i }}.</p>
        <div class="flex flex-row justify-center">
          <div v-for="n in 15" :key="n" class="flex w-6 flex-col">
            <p class="text-center">{{ n }}</p>
            <p class="text-center">{{ permutation[0][n - 1] }}</p>
          </div>
        </div>
        <p class="text-center">
          {{ permutation[1] }}
        </p>
      </div>
    </div>
    <div v-if="showMoves">
      <p v-for="move in movesList" :key="move" class="text-center">
        {{ move }}
      </p>
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
const movesList = ref([]) as Ref<string[]>;
const transpositions = ref([]) as Ref<string[]>;
const showMoves = ref(false);
const showGroupTheory = ref(false);
const classicMode = ref(false);
const transpositionString = ref('');
const permutations = ref([
  [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15], ''],
]) as Ref<[number[], string][]>;

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
  transpositions.value = [];
  transpositionString.value = '';
  permutations.value = [
    [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15], ''],
  ];
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
  transpositions.value = [];
  transpositionString.value = '';
  permutations.value = [];
  updatePermutations();
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
  transpositions.value.unshift(`(0, ${temp})`);
  transpositionString.value = `(0, ${temp})` + transpositionString.value;
  updatePermutations();
}

function moveDown() {
  if (zeroLocation[0] === 3) return;
  const temp = grid.value[zeroLocation[0] + 1][zeroLocation[1]];
  grid.value[zeroLocation[0] + 1][zeroLocation[1]] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[0] += 1;
  moves.value += 1;
  movesList.value.unshift('down');
  transpositions.value.unshift(`(0, ${temp})`);
  transpositionString.value = `(0, ${temp})` + transpositionString.value;
  updatePermutations();
}

function moveLeft() {
  if (zeroLocation[1] === 0) return;
  const temp = grid.value[zeroLocation[0]][zeroLocation[1] - 1];
  grid.value[zeroLocation[0]][zeroLocation[1] - 1] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[1] -= 1;
  moves.value += 1;
  movesList.value.unshift('left');
  transpositions.value.unshift(`(0, ${temp})`);
  transpositionString.value = `(0, ${temp})` + transpositionString.value;
  updatePermutations();
}

function moveRight() {
  if (zeroLocation[1] === 3) return;
  const temp = grid.value[zeroLocation[0]][zeroLocation[1] + 1];
  grid.value[zeroLocation[0]][zeroLocation[1] + 1] = 0;
  grid.value[zeroLocation[0]][zeroLocation[1]] = temp;
  zeroLocation[1] += 1;
  moves.value += 1;
  movesList.value.unshift('right');
  transpositions.value.unshift(`(0, ${temp})`);
  transpositionString.value = `(0, ${temp})` + transpositionString.value;
  updatePermutations();
}

onKeyStroke(
  ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'r', 's'],
  (e) => {
    if (isSolving) {
      return;
    }
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

function updatePermutations() {
  if (zeroLocation[0] === 3 && zeroLocation[1] === 3) {
    const gridArray = [];
    for (const row of grid.value) {
      for (const tile of row) {
        gridArray.push(tile);
      }
    }
    permutations.value.push([gridArray, transpositionString.value]);
    transpositionString.value = '';
  }
}

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

// UTILITY FUNCTIONS

function toIndex(row: number, column: number) {
  // row, column
  return row * 4 + column;
}

function copy(arr: any[]) {
  return JSON.parse(JSON.stringify(arr));
}

function invert(seq: string[]) {
  const inv = { right: 'left', left: 'right', up: 'down', down: 'up' } as {
    [key: string]: string;
  };
  //    let inv = {"left": "up", "up":"left", "down":"right", "right":"down"};
  const ret = [];
  for (const elem of seq) {
    ret.push(inv[elem]);
  }
  return ret;
}

// VARIABLES
// These were previously derived using separate code, but are hardcoded here to save on computation time

const solved = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 'empty'];

// Sequences of moves which move whatever tile is in index n to index 10 where it can be cycled easily
const rho = {
  0: [
    'up',
    'up',
    'up',
    'left',
    'left',
    'left',
    'down',
    'right',
    'up',
    'right',
    'down',
    'left',
    'down',
    'right',
    'up',
    'right',
    'down',
    'down',
  ],
  1: [
    'up',
    'up',
    'up',
    'left',
    'left',
    'down',
    'right',
    'up',
    'left',
    'down',
    'down',
    'right',
    'up',
    'right',
    'down',
    'down',
  ],
  2: [
    'left',
    'up',
    'up',
    'up',
    'left',
    'down',
    'down',
    'down',
    'right',
    'up',
    'up',
    'left',
    'down',
    'down',
    'right',
    'right',
  ],
  3: [
    'up',
    'up',
    'up',
    'left',
    'down',
    'down',
    'right',
    'up',
    'left',
    'down',
    'right',
    'down',
  ],
  4: [
    'left',
    'left',
    'left',
    'up',
    'up',
    'right',
    'down',
    'left',
    'up',
    'right',
    'right',
    'down',
    'left',
    'down',
    'right',
    'right',
  ],
  5: [
    'up',
    'up',
    'left',
    'left',
    'down',
    'right',
    'up',
    'right',
    'down',
    'down',
  ],
  6: ['left', 'up', 'up', 'left', 'down', 'down', 'right', 'right'],
  7: ['up', 'up', 'left', 'down', 'right', 'down'],
  8: [
    'up',
    'left',
    'left',
    'left',
    'up',
    'right',
    'right',
    'right',
    'down',
    'left',
    'left',
    'up',
    'right',
    'right',
    'down',
    'down',
  ],
  9: ['up', 'left', 'left', 'up', 'right', 'right', 'down', 'down'],
  10: [],
  12: [
    'left',
    'left',
    'left',
    'up',
    'right',
    'right',
    'down',
    'left',
    'up',
    'right',
    'down',
    'right',
  ],
  13: ['left', 'left', 'up', 'right', 'down', 'right'],
} as { [key: number]: string[] };

// Permutation (11 14 n)
const forwardCycle = ['up', 'left', 'down', 'right'];

// FUNCTIONS
function swap(idx: number) {
  // cycling idx and the tiles in positions 11 and 14 (which will work out to be the "12" and "15" tiles on a solved puzzle)
  // (11 14 idx)
  try {
    return rho[idx]
      .concat(copy(forwardCycle))
      .concat(invert(copy(rho[idx]).reverse()));
  } catch {
    throw new Error(
      'Specified permutation is a 2-cycle (impossible in this puzzle)',
    );
  }
}

function to3Cycles(cycles: number[][]) {
  const evenCycles = [];
  const oddCycles = [];
  const final = [];
  for (const c of cycles) {
    if (c.length % 2 === 0) {
      evenCycles.push(c);
    } else {
      oddCycles.push(c);
    }
  }
  if (evenCycles.length % 2 === 1) {
    throw new Error(`${cycles} is not an even permutation!`);
  }
  for (const c of oddCycles) {
    for (let i = 0; i < c.length - 1; i += 2) {
      final.push([c[i], c[i + 1], c[i + 2]]);
    }
  }

  for (let i = 0; i < evenCycles.length; i += 2) {
    const c1 = evenCycles[i];
    const c2 = evenCycles[i + 1];
    for (let j = 0; j < c1.length - 3; j += 2) {
      final.push([c1[j], c1[j + 1], c1[j + 2]]);
    }
    final.push([c1[c1.length - 2], c2[0], c1[c1.length - 1]]);
    final.push([c1[c1.length - 2], c2[0], c2[1]]);
    for (let j = 1; j < c2.length - 1; j += 2) {
      final.push([c2[j], c2[j + 1], c2[j + 2]]);
    }
  }
  return final;
}

function cycleToMoves(cycle: number[]) {
  // takes in a 3-cycle and outputs moves to solve it
  if (cycle.includes(11) && cycle.includes(14)) {
    for (let i = 0; i < cycle.length; i++) {
      if (cycle[i] !== 11 && cycle[i] !== 14) {
        const ret = swap(cycle[i]);
        if (cycle[(i + 1) % cycle.length] === 11) {
          return ret;
        } else {
          return ret.concat(ret);
        }
      }
    }
  } else if (cycle.includes(11) && !cycle.includes(14)) {
    if (cycle[1] === 11) {
      cycle = [11, cycle[2], cycle[0]];
    } else if (cycle[2] === 11) {
      cycle = [11, cycle[0], cycle[1]];
    }
    // [11, 14, cycle[1]], [11, 14, cycle[1]], [11, 14, cycle[2]]
    return swap(cycle[1]).concat(swap(cycle[1])).concat(swap(cycle[2]));
  } else if (cycle.includes(14) && !cycle.includes(11)) {
    if (cycle[1] === 14) {
      cycle = [14, cycle[2], cycle[0]];
    } else if (cycle[2] === 14) {
      cycle = [14, cycle[0], cycle[1]];
    }
    // [11, 14, cycle[2]], [11, 14, cycle[2]], [11, 14, cycle[1]]
    return swap(cycle[1]).concat(swap(cycle[2])).concat(swap(cycle[2]));
  } else {
    return cycleToMoves([11, cycle[0], cycle[1]]).concat(
      cycleToMoves([11, cycle[2], cycle[0]]),
    );
  }
}

function toMoves(permutation: number[] | string[]) {
  // takes in a scramble and outputs moves to solve it
  const done = [] as number[];
  const cycles = [];
  for (let i = 0; i < permutation.length; i++) {
    if (permutation[i] === 'empty') {
      continue;
    }
    if (!done.includes(permutation[i])) {
      done.push(permutation[i]);
      if (permutation[i] !== i) {
        cycles.push([permutation[i]]);
        let current = permutation[i];
        while (current !== i) {
          current = permutation[current];
          done.push(current);
          cycles[cycles.length - 1].push(current);
        }
      }
    }
  }
  let moves = [] as string[];
  for (const c of to3Cycles(cycles).reverse()) {
    moves = moves.concat(cycleToMoves(c));
  }
  return moves;
}

function emptyToBottomRight(board) {
  board = copy(board);
  const moves = [];
  const start = board.indexOf('empty');
  let currentRow = Math.floor(start / 4);
  let currentCol = start % 4;
  for (let i = 0; i < 3 - (start % 4); i++) {
    moves.push('right');
    board[toIndex(currentRow, currentCol)] =
      board[toIndex(currentRow, currentCol + 1)];
    board[toIndex(currentRow, currentCol + 1)] = 'empty';
    currentCol++;
  }
  for (let i = 0; i < 3 - Math.floor(start / 4); i++) {
    moves.push('down');
    board[toIndex(currentRow, currentCol)] =
      board[toIndex(currentRow + 1, currentCol)];
    board[toIndex(currentRow + 1, currentCol)] = 'empty';
    currentRow++;
  }
  return [moves, board];
}

function optimizeMoves(moves: string[]) {
  const inv = { right: 'left', left: 'right', up: 'down', down: 'up' } as {
    [key: string]: string;
  };
  for (let i = 0; i < 100; i++) {
    let hasChanged = false;
    let j = 0;
    while (j < moves.length) {
      if (moves[j] === inv[moves[j + 1]]) {
        moves.splice(j, 2);
        hasChanged = true;
      }
      j++;
    }
    if (!hasChanged) {
      break;
    }
  }
  return moves;
}

function solve(grid: number[][]) {
  let board = [];
  for (const row of grid) {
    for (const elem of row) {
      if (elem === 0) {
        board.push('empty');
      } else {
        board.push(elem - 1);
      }
    }
  }
  const ret = emptyToBottomRight(board);
  board = ret[1];
  const moves = ret[0].concat(toMoves(board));
  return optimizeMoves(moves);
}

const moveFuncs = {
  left: moveLeft,
  right: moveRight,
  up: moveUp,
  down: moveDown,
};

let isSolving = false;

function iterMoves(moves: string[]) {
  if (moves.length === 0) {
    isSolving = false;
    return;
  }
  moveFuncs[moves[0]]();
  setTimeout(iterMoves, 100, moves.slice(1));
}

function solveButton() {
  if (isSolving) {
    return;
  }
  isSolving = true;
  iterMoves(solve(grid.value));
}
</script>

<style scoped>
.grid-transition-move {
  transition: transform 0.25s ease-in-out;
}
</style>
