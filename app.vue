<template>
<div class="flex justify-between bg-gray-900">

  <!-- LEFT: LED MATRIX -->
  <div class="border-r">
    <div class="led-screen rounded m-2">
      <div
        v-for="(row, rowIndex) in viewScreen"
        :key="rowIndex"
        class="led-row"
      >
        <span
          v-for="(dot, colIndex) in row"
          :key="colIndex"
          class="led-dot"
          :class="{
            on: dot,
            'led-red': dot && red,
            'led-green': dot && !red
          }"
          @mousedown="startDraw(rowIndex, colIndex)"
          @mouseenter="draw(rowIndex, colIndex)"
        ></span>
      </div>
    </div>

    <!-- BUTTONS -->
    <div class="flex flex-wrap">
      <button @click="saveScreen" class="btn">save</button>
      <button @click="copyText" class="btn">copy</button>
      <button @click="fillScreen" class="btn">fill</button>
      <button @click="clearScreen" class="btn">reset</button>
      <button @click="red = !red" class="btn">
        {{ red ? 'Green' : 'Red' }}
      </button>
      <button @click="flipScreen90" class="btn">flip 90°</button>
    </div>
  </div>

  <!-- CENTER: HEX -->
  <div class="p-2">
    <p class="whitespace-pre-line font-semibold text-left text-xl text-white">
      {{ main_hex }}
    </p>
  </div>

  <!-- RIGHT: SAVED SCREENS -->
<div class="h-screen border-l overflow-y-hidden p-1 rotate-container">
  <h3 class="text-left px-2">Last Saved Screens</h3>

  <div class="grid grid-cols-2 gap-1 overflow-y-auto">
    <div
      v-for="(s, i) in  savedScreens.slice().reverse()"
      :key="i"
      class="small-led-screen"
      @click="loadSaved(s)"
    >
      <div v-for="(r, ri) in rotateScreenMinus90(s)" :key="ri" class="led-row">
        <span
          v-for="(d, ci) in r"
          :key="ci"
          class="small-led-dot"
          :class="{ on: d }"
        ></span>
      </div>
    </div>
  </div>
</div>


</div>
</template>

<script>
export default {
  name: 'HomeView',

  data() {
    return {
      red: false,

      ROWS: 16,
      COLS: 32,

      rotation: 0,

      baseScreen: [],
      viewScreen: [],

      isDrawing: false,
      main_hex: '',
      savedScreens: []
    }
  },

  created() {
    this.initScreens()
    this.savedScreens = JSON.parse(localStorage.getItem('led-screens') || '[]')
    window.addEventListener('mouseup', this.stopDraw)
  },

  mounted(){
    this.flipScreen90()
    this.flipScreen90()
    this.flipScreen90()
  },

  beforeUnmount() {
    window.removeEventListener('mouseup', this.stopDraw)
  },

  methods: {
    /* ---------- INIT ---------- */
rotateScreenMinus90(screen) {
  const rowCount = screen.length;
  const colCount = screen[0].length;
  const rotated = [];

  for (let c = colCount - 1; c >= 0; c--) {
    const newRow = [];
    for (let r = 0; r < rowCount; r++) {
      newRow.push(screen[r][c]);
    }
    rotated.push(newRow);
  }

  return rotated;
},


    initScreens() {
      this.baseScreen = Array.from({ length: this.ROWS }, () =>
        Array.from({ length: this.COLS }, () => false)
      )
      this.viewScreen = this.clone(this.baseScreen)
      this.generateHex()
    },

    clone(m) {
      return m.map(r => r.slice())
    },

    /* ---------- DRAWING ---------- */

    startDraw(r, c) {
      this.isDrawing = true
      this.draw(r, c)
    },

    stopDraw() {
      this.isDrawing = false
    },

    draw(r, c) {
      if (!this.isDrawing) return
      this.toggle(r, c)
    },

    toggle(r, c) {
      this.viewScreen[r][c] = !this.viewScreen[r][c]

      const [br, bc] = this.viewToBase(r, c)
      this.baseScreen[br][bc] = this.viewScreen[r][c]

      this.generateHex()
    },

    /* ---------- COORD MAP ---------- */

    viewToBase(r, c) {
      const R = this.baseScreen.length
      const C = this.baseScreen[0].length

      switch (this.rotation) {
        case 0: return [r, c]
        case 1: return [R - 1 - c, r]
        case 2: return [R - 1 - r, C - 1 - c]
        case 3: return [c, C - 1 - r]
      }
    },

    /* ---------- ROTATION ---------- */

    flipScreen90() {
      this.rotation = (this.rotation + 1) % 4
      this.viewScreen = this.rotateCW(this.viewScreen)
    },

    rotateCW(m) {
      return m[0].map((_, i) =>
        m.map(row => row[i]).reverse()
      )
    },

    /* ---------- HEX ---------- */

    generateHex() {
      let out = []

      for (let r = 0; r < this.baseScreen.length; r++) {
        let acc = 0
        let bits = 0
        let rowHex = ''

        for (let c = 0; c < this.baseScreen[0].length; c++) {
          acc = (acc << 1) | (this.baseScreen[r][c] ? 1 : 0)
          bits++

          if (bits === 4) {
            rowHex += acc.toString(16).toUpperCase()
            acc = 0
            bits = 0
          }
        }
        out.push(`0x${rowHex}`)
      }
      
      this.main_hex = ""
      for (var i = 0; i < out.length; i++) {
        //alert(i)
        this.main_hex += out[i] +", " 
        if(i==7){
          this.main_hex +=  '\n'
        }
      }

      //this.main_hex = out.join(', ')
    },

    /* ---------- STORAGE ---------- */

    saveScreen() {
      this.savedScreens.push(this.clone(this.baseScreen))
      localStorage.setItem('led-screens', JSON.stringify(this.savedScreens))
    },

    loadSaved(s) {
      this.baseScreen = this.clone(s)
      this.viewScreen = this.clone(s)
      //this.rotation = 0
      this.generateHex()
    },

    /* ---------- UTILS ---------- */

    clearScreen() {
      this.initScreens()
    },

    fillScreen() {
      this.baseScreen = this.baseScreen.map(r =>
        r.map(() => true)
      )
      this.viewScreen = this.clone(this.baseScreen)
      this.generateHex()
    },

    copyText() {
      navigator.clipboard.writeText(this.main_hex)
    }
  }
}
</script>

<style>
.led-screen {
  background: #222;
  padding: 5px;
}

.small-led-screen {
  background: #222;
  padding: 6px;
}

.led-row {
  display: flex;
}

.led-dot {
  width: 10px;
  height: 10px;
  margin: 2px;
  border-radius: 50%;
  background: #444;
  cursor: pointer;
}

.small-led-dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: #444;
}

.on {
  background: #00ff55;
}

.led-red {
  background: red;
}

.led-green {
  background: #00ff55;
}

.btn {
  margin: 4px;
  padding: 4px 8px;
  background: #111;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}
</style>
