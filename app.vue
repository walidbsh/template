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
        :class="['led-dot', { 'on': dot === 1 }]"
        
      ></span>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const rows = 32;
    const cols = 16;

    // create 2D array filled with 0
    const screen = ref(Array.from({ length: rows }, () => new Array(cols).fill(0)));

    // Example: turn some dots on

    return { screen };
  },
  methods:{

  }
}
</script>

<style>
.led-screen {
  display: inline-block;
  background: #222;
  padding: 10px;
  border-radius: 8px;
}

.led-row {
  display: flex;
}

.led-dot {
  width: 15px;
  height: 15px;
  margin: 2px;
  border-radius: 50%;
  background-color: #444; /* off state */
  transition: background-color 0.2s;
}

.led-dot.on {
  background-color: #0f0; /* on state */
}
</style>
