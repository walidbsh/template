<template>
  <div class="led-screen">
    <div
      v-for="(row, rowIndex) in screen"
      :key="rowIndex"
      class="led-row"
    >
      <span
        v-for="(dot, colIndex) in row"
        :key="colIndex"
        :class="[
          'led-dot',
          { 
            on: dot === 1,
            selected: rowIndex === selectedRow && colIndex === selectedCol,
            hovered: rowIndex === hoverRow && colIndex === hoverCol
          }
        ]"
        @mousedown="startDraw(rowIndex, colIndex)"
        @mouseenter="draw(rowIndex, colIndex)"
        @mouseup="stopDraw"
      ></span>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';

export default {
  setup() {
    const rows = 10;
    const cols = 16;

    // 2D LED array
    const screen = ref(
      Array.from({ length: rows }, () => new Array(cols).fill(0))
    );

    // state
    const isDrawing = ref(false);
    const selectedRow = ref(null);
    const selectedCol = ref(null);
    const hoverRow = ref(null);
    const hoverCol = ref(null);

    const toggle = (row, col) => {
      screen.value[row][col] = screen.value[row][col] ? 0 : 1;
    };

    const startDraw = (row, col) => {
      isDrawing.value = true;
      toggle(row, col);
      selectedRow.value = row;
      selectedCol.value = col;
    };

    const draw = (row, col) => {
      hoverRow.value = row;
      hoverCol.value = col;
      if (!isDrawing.value) return;
      toggle(row, col);
    };

    const stopDraw = () => {
      isDrawing.value = false;
    };

    onMounted(() => {
      window.addEventListener('mouseup', stopDraw);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('mouseup', stopDraw);
    });

    return {
      screen,
      startDraw,
      draw,
      stopDraw,
      selectedRow,
      selectedCol,
      hoverRow,
      hoverCol
    };
  }
};
</script>

<style>
.led-screen {
  display: inline-block;
  background: #222;
  padding: 10px;
  user-select: none;
}

.led-row {
  display: flex;
}

.led-dot {
  width: 18px;
  height: 18px;
  margin: 2px;
  border-radius: 50%;
  background: #444;
  cursor: pointer;
}

/* LED ON */
.led-dot.on {
  background: #00ff55;
}

/* Hover preview */
.led-dot.hovered:not(.selected) {
  outline: 1px dashed #888;
  outline-offset: 2px;
}

/* Selected LED */
/*.led-dot.selected {
  outline: 2px solid yellow;
  outline-offset: 2px;
}*/
</style>
