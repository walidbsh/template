<template>
<div class="flex justify-between">
    
  <div class="border-r">
    <div class="led-screen rounded m-2">
      <div
        v-for="(row, rowIndex) in screen"
        :key="rowIndex"
        class="led-row"
      >
        <span
          v-for="(dot, colIndex) in row"
          :key="colIndex"
          class="led-dot"
          :class="{
            on: dot,
            'led-red': dot && red,     // red if red = true and dot is on
            'led-green': dot && !red,  // green if red = false and dot is on
            selected: rowIndex === selectedRow && colIndex === selectedCol,
            hovered: rowIndex === hoverRow && colIndex === hoverCol
          }"
          @mousedown="startDraw(rowIndex, colIndex)"
          @mouseenter="draw(rowIndex, colIndex)"
        ></span>
      </div>
    </div>

    <div class="flex">
      <button @click="saveScreen" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">save</button>
      <button v-if="false" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">load</button>
      <button @click="copyText" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">copy</button>
      <button @click="fill_screen" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">fill</button>
      <button @click="clear_screen" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">rest</button>
      <button v-if="red" @click="red=!red" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">Green</button>
      <button v-else @click="red=!red" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold">Red</button>
      <button @click="flipScreen90" class="p-1 px-2 bg-gray-900 rounded mx-2 text-white font-bold whitespace-nowrap">flip 90°</button>
    </div>
  </div>





  <div class="p-2">
    <!-- <textarea :value="main_hex"></textarea> -->
    <p v-if="true" class=" whitespace-pre-line mb-2 font-semibold text-left">{{main_hex}}</p> 
  </div>


  <div class="h-screen border-l overflow-y-hidden p-1">
    
    <h3 class="text-left text-white px-2">Last Saved Screens</h3>

    <div class="rounded grid grid-cols-2 gap-1 auto-rows-auto h-full overflow-y-auto">
      <div
        v-for="(s, si) in savedScreens.slice().reverse()"
        :key="si"
        class="small-led-screen"
        @click="screen =  s.map(row => [...row])"
      >
        <div
          v-for="(rrr, rowIndex) in s"
          :key="rowIndex"
          class="led-row"
        >
          <span
            v-for="(dot, colIndex) in rrr"
            :key="colIndex"
            class="small-led-dot"
            :class="{ on: dot }"
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
      red:false,
      rows: 16,
      cols: 32,

      // ✅ REACTIVE LED MATRIX
      screen: [],

      // state
      isDrawing: false,
      selectedRow: null,
      selectedCol: null,
      hoverRow: null,
      hoverCol: null,
      main_hex:'',
      words:[],
      savedScreens:[]
    }
  },

  created() {
    // create 32 x 16 matrix (cols x rows)
    this.screen = Array.from({ length: this.cols }, () =>
      Array.from({ length: this.rows }, () => false)
    )
  },

  mounted() {
    window.addEventListener('mouseup', this.stopDraw)
    this.isDrawing = true
    this.draw(0, 0)
    this.draw(0, 0)
    this.isDrawing = false
    this.savedScreens = JSON.parse(localStorage.getItem('led-screens') || '[]')
  },

  beforeUnmount() {
    window.removeEventListener('mouseup', this.stopDraw)
  },

  methods: {
    copyText() {

      // Modern approach
      navigator.clipboard.writeText(this.main_hex)
        .then(() => {
          alert('Text copied to clipboard!');
        })
        .catch(err => {
          console.error('Failed to copy: ', err);
        });
    },
    flipScreen90() {
      const newScreen = this.screen[0].map((_, colIndex) =>
        this.screen.map(row => row[colIndex]).reverse()
      );

      // Swap rows and cols
      const oldRows = this.rows;
      this.rows = this.cols;
      this.cols = oldRows;

      this.screen = newScreen;
    }
    ,
    clear_screen(){
      this.screen = Array.from({ length: this.cols }, () =>
      Array.from({ length: this.rows }, () => false)
    )
    },
    fill_screen(){
      this.screen = Array.from({ length: this.cols }, () =>
      Array.from({ length: this.rows }, () => true)
    )
    },
    saveScreen() {
      const all = JSON.parse(
        localStorage.getItem('led-screens') || '[]'
      )

      // deep copy current screen
      all.push(JSON.parse(JSON.stringify(this.screen)))

      localStorage.setItem(
        'led-screens',
        JSON.stringify(all)
      )

      this.savedScreens = JSON.parse(localStorage.getItem('led-screens') || '[]')

    },
    loadScreen(index) {
      const all = JSON.parse(
        localStorage.getItem('led-screens') || '[]'
      )

      if (!all[index]) return

      this.screen = JSON.parse(JSON.stringify(all[index]))
    },
    saveAllScreens(data) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data))
    },
    decToHex(n) { 
      const hexTable = ['0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F']
      return hexTable[n] 
    },
    toggle(row, col) {
      this.screen[row][col] = !this.screen[row][col]
    },

    startDraw(row, col) {
      this.isDrawing = true
      //this.toggle(row, col)
      //alert('clicked')
      this.draw(row, col)
    },

    draw(row, col) {
      this.hoverRow = row
      this.hoverCol = col

      if (!this.isDrawing) return
      this.toggle(row, col)

      let s = ''
      let vertical_lengh = 4
      let vertical_acc = 0 
      this.words = []

      for (var j = 0; j < this.rows; j++){ 
        for (var i = 0; i < this.cols ; i++) { 
          vertical_lengh-- 
          let p = (this.cols-i-1) % 4 
          vertical_acc += Math.pow(2,p)*(this.screen[i][j]==0?0:1) 

          if(vertical_lengh==0){ 
            vertical_lengh = 4 
            s = this.decToHex(vertical_acc) + s; 
            vertical_acc = 0 
          } 
        } 
        this.words.push(s) 
        s = '' 
      } 
      console.log('start hexign')
      this.main_hex = '' 
      for (var i = 0; i < this.words.length; i++) { 
        this.main_hex += '0x' + this.words[i] + ', '
        if((i+1)==((this.words.length)/2)) {
          this.main_hex += '\n'
        } 
      }
      console.log('end hexign')
      /*console.log(this.main_hex)*/


    },

    stopDraw() {
      this.isDrawing = false
    }
  }
}
</script>

<style>
.led-screen {
  display: inline-block;
  background: #222;
  padding: 5px;
  user-select: none;
}

.small-led-screen {
  display: inline-block;
  background: #222;
  padding: 10px;
  user-select: none;
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
  cursor: pointer;
}

.led-dot.on {
  /*background: #00ff55;*/
}

.small-led-dot.on {
  background: #00ff55;
  opacity:50%;
}


.led-red {
  background-color: #ff0000;
}

.led-green {
  background-color: #00ff55;
}


.led-dot.hovered:not(.selected) {
  outline: 1px dashed #888;
  outline-offset: 2px;
}

</style>
