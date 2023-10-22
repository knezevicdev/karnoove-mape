<template>
  <div class="container  select-none mx-auto">
    <h1 class="text-3xl font-bold text-center">
      Karnoove Mape
    </h1>
    <div class="grid grid-cols-4 mt-6 mb-2 gap-3">
      <label>
        <input type="checkbox" v-model="showNumbers" >
        Prikaži brojeve
      </label>
      <label>
        <input type="checkbox" v-model="useDontCares" >
        Nebitna stanja
      </label>
      <select v-model="type">
        <option :value="Type.fourVariables">4 promenljive</option>
        <option :value="Type.fiveVariables">5 promenljivih</option>
        <option :value="Type.sixVariables">6 promenljivih</option>
      </select>
      <button
          class="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded"
          @click="restart"
      >Restart</button>
    </div>
    <div
        class="grid relative"
        :class="{
          'grid-cols-1': type === Type.fourVariables,
          'grid-cols-2': type === Type.fiveVariables,
          'grid-cols-11': type === Type.sixVariables
        }"
        ref="table"
    >
      <div
          class="grid text-center border border-black"
          :class="{
            'grid-cols-6': isSixColumn(gridIndex),
            'grid-cols-5': !isSixColumn(gridIndex),
            'col-span-5': type === Type.sixVariables && gridIndex % 2 === 1,
            'col-span-6': type === Type.sixVariables && gridIndex % 2 === 0,
            'border-l-0': type !== Type.fourVariables && gridIndex % 2 === 0,
            'border-r-0': type !== Type.fourVariables && gridIndex % 2 === 1,
            'border-t-0': type === Type.sixVariables && gridIndex > 2,
            'border-b-0': type === Type.sixVariables && gridIndex < 3,
          }"
          v-for="gridIndex in numberOfGrids"
      >
        <template v-if="type !== Type.fourVariables && gridIndex < 3">
          <div class="bg-blue-100 border border-black"></div>
          <div class="bg-blue-100 col-span-4 border border-black">t{{ gridIndex === 1 ? "'" : '' }}</div>
          <div v-if="isSixColumn(gridIndex)" class="bg-blue-100 border border-black"></div>
        </template>
        <div class="bg-blue-100 p-2 border border-black">
          {{ gridIndex < 3 ? 'XY\\ZW' : '' }}
        </div>
        <div v-for="schema in binarySchema" class="bg-blue-100 p-2 border border-black" :class="gridIndex > 2 ? 'py-3' : ''">
          {{ gridIndex < 3 ? schema : '' }}
        </div>
        <div v-if="isSixColumn(gridIndex)" class="bg-blue-100 border border-black"></div>
        <template v-for="index in 20" :key="index">
          <div class="border border-black">
            <div v-if="index % 5 === 1" class="bg-blue-100 p-2">
              {{ binarySchema[(index - 1) / 5] }}
            </div>
            <div
                v-else
                class="p-2 cursor-pointer relative"
                :class="{
                  'bg-amber-100': getValue(index, gridIndex - 1) === 1,
                  'bg-amber-50': getValue(index, gridIndex - 1) === '?'
                }"
                @click="toggleBit(index, gridIndex - 1)"
            >
              {{ getValue(index, gridIndex - 1) }}
              <div v-if="showNumbers" class="absolute top-0.5 right-1 text-xs font-semibold">{{ getNumber(index, gridIndex) }}</div>
            </div>
          </div>
          <div v-if="isSixColumn(gridIndex) && index === 5" class="bg-blue-100 border border-black row-span-4 grid place-items-center">
            v{{ gridIndex === 2 ? "'" : ''}}
          </div>
        </template>
      </div>
      <canvas
          class="absolute -top-2 -left-2"
          :class="{
            'pointer-events-none': selectedGroupIndex === -1,
            'cursor-cell': selectedGroupIndex !== -1
          }"
          ref="canvas"
          @mousedown="canvasMouseDown"
          @mouseup="canvasMouseUpOrOut"
          @mouseout="canvasMouseUpOrOut"
          @mousemove="canvasMouseMove"
      ></canvas>
    </div>
    <div class="flex gap-6 mt-4">
      <div>
        <button
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
            @click="addGroup"
        >Dodaj grupu (pravougaonik)</button>
        <p class="mt-2" v-if="groups.length > 0">
          - Kliknite na pravougaonik kako biste ušli u režim crtanja. <br />
          - Kliknite i povucite preko tabele da biste nacrtali pravougaonik. <br />
          - Klikom na odabrani pravougaonik izlazite iz režima crtanja.
        </p>
      </div>
      <div class="top-2">
        <div
          v-for="(group, groupIndex) in groups"
          :key="group.color"
          class="flex gap-1"
        >
          <div class="grid grid-cols-5 gap-0.5 w-60">
            <button
              class="flex items-center p-4 col-span-4 border-4"
              :class="selectedGroupIndex === groupIndex ? 'border-black' : 'border-white'"
              :style="`background-color: ${group.color}`"
              @click="setSelectedGroupIndex(groupIndex)"
            >
            </button>
            <button @click="deleteGroup(groupIndex)">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0" />
              </svg>
            </button>
          </div>
          <input type="text" placeholder="f" class="my-0.5">
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {computed, onMounted, reactive, ref, watch, watchEffect} from "vue";

const predefinedColors = [
  "#ff0000",
  "#0074D9",
  "#FF851B",
  "#2ECC40",
  "#85144b",
  "#FFDC00",
  "#39CCCC",
  "#FF4136",
  "#B10DC9",
];

type CellValue = 0 | 1 | '?';
type Group = {
  color: string;
  rectangles: {
    x: number;
    y: number;
    width: number;
    height: number;
  }[];
}

enum Type {
  fourVariables,
  fiveVariables,
  sixVariables,
}

const binarySchema = ['00', '01', '11', '10'];

const showNumbers = ref(false);
const useDontCares = ref(false);

const type = ref<Type>(Type.fourVariables);
const grid = reactive([
  Array(16).fill(0),
  Array(16).fill(0),
  Array(16).fill(0),
  Array(16).fill(0),
]);
const groups = reactive<Group[]>([]);
const selectedGroupIndex = ref(-1);

const numberOfGrids = computed(() => {
  switch (type.value) {
    case Type.fourVariables:
      return 1;
    case Type.fiveVariables:
      return 2;
    case Type.sixVariables:
      return 4;
  }
});

const getRowAndCol = (index: number) => {
  const row = Math.floor((index - 1) / 5);
  const col = (index - 1) % 5 - 1;
  const flatIndex = row * 4 + col;

  return {
    row, col, flatIndex
  }
}

const getNumber = (index: number, gridIndex: number) => {
  const { row, col } = getRowAndCol(index);

  return parseInt(`${binarySchema[row]}${binarySchema[col]}` , 2) + (gridIndex - 1) * 16;
}

const toggleBit = (index: number, gridIndex: number) => {
  const { flatIndex } = getRowAndCol(index);

  const value = grid[gridIndex][flatIndex];
  let nextValue: CellValue = 0;
  if (value === 0) nextValue = 1;
  if (useDontCares.value && value === 1) nextValue = '?';

  grid[gridIndex][flatIndex] = nextValue;
}

watchEffect(() => {
  if (!useDontCares.value) {
    grid.forEach((row) => {
      row.forEach((cell, index) => {
        if (cell === '?') row[index] = 0;
      })
    })
  }
})

watch(() => type.value, () => {
  setTimeout(() => {
    paintRects();
  }, 0);
});

onMounted(() => {
  window.addEventListener('resize', () => {
    paintRects();
  });
});

const getValue = (index: number, gridIndex: number) => {
  const { flatIndex } = getRowAndCol(index);

  return grid[gridIndex][flatIndex];
}

const isSixColumn = (gridIndex: number) => {
  return type.value === Type.sixVariables && gridIndex % 2 === 0;
}

const addGroup = () => {
  const color = predefinedColors.find((color) => !groups.find((group) => group.color === color)) || `#${Math.floor(Math.random()*16777215).toString(16)}`;

  groups.push({
    color,
    rectangles: [],
  });
}

const setSelectedGroupIndex = (groupIndex: number) => {
  if (groupIndex === selectedGroupIndex.value) {
    selectedGroupIndex.value = -1;
  } else {
    selectedGroupIndex.value = groupIndex;
  }

  paintRects();
}

const deleteGroup = (groupIndex: number) => {
  groups.splice(groupIndex, 1);

  if (groups.length === 0 || groupIndex === selectedGroupIndex.value) {
    selectedGroupIndex.value = -1;
  } else if (selectedGroupIndex.value >= groupIndex) {
    selectedGroupIndex.value -= 1;
  }

  paintRects();
}

const table = ref<HTMLDivElement>();
const canvas = ref<HTMLCanvasElement>();
const canvasContext = ref<CanvasRenderingContext2D | null>(null);
const canvasOffset = ref({ x: 0, y: 0 });
const startPosition = ref({ x: 0, y: 0 });
const endPosition = ref<{x: number, y: number} | null>(null);
const isMouseDown = ref(false);

const paintRects = () => {
  if (!canvasContext.value) {
    canvasContext.value = canvas.value!.getContext('2d')!;
  }

  const tableBoundingRect = table.value!.getBoundingClientRect();

  canvas.value!.width = tableBoundingRect.width + 16;
  canvas.value!.height = tableBoundingRect.height + 16;

  canvasContext.value.clearRect(0, 0, canvas.value!.width, canvas.value!.height);

  groups.forEach((group) => {
    canvasContext.value!.strokeStyle = group.color;
    canvasContext.value!.lineWidth = 3;
    group.rectangles.forEach((rectangle) => {
      canvasContext.value!.strokeRect(rectangle.x, rectangle.y, rectangle.width, rectangle.height);
    })
  })

  if (endPosition.value) {
    canvasContext.value!.strokeStyle = groups[selectedGroupIndex.value].color;
    canvasContext.value!.lineWidth = 3;
    canvasContext.value!.strokeRect(
      Math.min(startPosition.value.x, endPosition.value.x),
      Math.min(startPosition.value.y, endPosition.value.y),
      Math.abs(startPosition.value.x - endPosition.value.x),
      Math.abs(startPosition.value.y - endPosition.value.y),
    );
  }
}

const canvasMouseDown = (event: MouseEvent) => {
  if (!canvas.value) return;

  const canvasBoundingRect = canvas.value.getBoundingClientRect();
  canvasOffset.value = {
    x: canvasBoundingRect.left,
    y: canvasBoundingRect.top,
  };

  startPosition.value = {
    x: parseInt(String(event.clientX - canvasOffset.value.x)),
    y: parseInt(String(event.clientY - canvasOffset.value.y)),
  };

  isMouseDown.value = true;
}

const canvasMouseUpOrOut = () => {
  if (!endPosition.value) return;

  groups[selectedGroupIndex.value].rectangles.push({
    x: Math.min(startPosition.value.x, endPosition.value.x),
    y: Math.min(startPosition.value.y, endPosition.value.y),
    width: Math.abs(startPosition.value.x - endPosition.value.x),
    height: Math.abs(startPosition.value.y - endPosition.value.y),
  });

  endPosition.value = null;
  isMouseDown.value = false;

  paintRects();
}

const canvasMouseMove = (event: MouseEvent) => {
  if (!canvas.value || !isMouseDown.value || selectedGroupIndex.value === -1) return;

  endPosition.value = {
    x: parseInt(String(event!.clientX - canvasOffset.value.x)),
    y: parseInt(String(event!.clientY - canvasOffset.value.y)),
  };

  paintRects();
}

const restart = () => {
  grid.forEach((row) => {
    row.fill(0);
  });
  groups.splice(0, groups.length);
  selectedGroupIndex.value = -1;

  paintRects();
}

</script>