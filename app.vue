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
export default {
  name: 'App',

  data() {
    const rows = 32;
    const cols = 16;

    return {
      rows,
      cols,

      // 2D LED array
      screen: Array.from({ length: rows }, () => new Array(cols).fill(0)),

      // state
      isDrawing: false,
      selectedRow: null,
      selectedCol: null,
      hoverRow: null,
      hoverCol: null,
    };
  },

  methods: {
    toggle(row, col) {
      this.screen[row][col] = this.screen[row][col] ? 0 : 1;
    },

    startDraw(row, col) {
      this.isDrawing = true;
      this.toggle(row, col);
      this.selectedRow = row;
      this.selectedCol = col;
    },

    draw(row, col) {
      this.hoverRow = row;
      this.hoverCol = col;

      if (!this.isDrawing) return;
      this.toggle(row, col);
    },

    stopDraw() {
      this.isDrawing = false;
    },
  },

  mounted() {
    window.addEventListener('mouseup', this.stopDraw);
  },

  beforeUnmount() {
    window.removeEventListener('mouseup', this.stopDraw);
  },
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
