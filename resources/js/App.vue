<template>
  <div class="flex items-center justify-center gap-4">
    <!-- <Button @click="play({ id: 'kick' })" label="Kick">🥁</Button>
    <Button @click="play({ id: 'hihat' })" label="Hi-hat">🎩</Button>
    <Button @click="play({ id: 'snare' })" label="Snare">🍗</Button>
    <Button @click="play({ id: 'cowbell' })" label="Cowbell">🔔</Button> -->

    <!-- black: [700, 1000],
      blue: [2200, 1000],
      brown: [3500, 1000],
      green: [5200, 1000],
      orange: [7100, 1000],
      pink: [8600, 1000],
      purple: [10000, 1000],
      red: [11600, 1000],
      white: [13300, 1000],
      yellow: [14700, 1000], -->
      <input 
        :value="spb" 
        @input="onInputTimeInterval" 
        placeholder="Інтервал в секундах між бісеринами">
      <input 
        :value="round"
        @input="onInputRound"
        placeholder="Окружність(довжина одного кола)">
      <input 
        :value="repeat"
        @input="onInputRepeat"
        placeholder="Повторів">
      <button @click="tooglePlay" :class="{ play: run }"> 
        <span v-if="!run"> ▶ </span>
        <span v-else > ❚❚ </span> 
      </button>
      <button @click="reset"> 
        <span> ◼ </span>
      </button>
      <v-stage :config="configKonva">
        <v-layer>
          <v-rect
            x="10"
            y="10"
            width="10"
            height="10"
            fill="black"
            stroke="black"
          >
          </v-rect>
          <v-rect
            v-for="item in bitmap"
            :config="getConfig(item)"
            @mousedown="beginDrawing" 
            @mouseover="keepDrawing" 
            @mouseup="stopDrawing"
            >
          </v-rect>
        </v-layer>
      </v-stage>
      <div>
        <div
          v-for="color in colors"
          class="color-selector"
          :style="{ backgroundColor : color }"
          @click="selectColor(color)"
        >
        </div>
      </div>
      <div class="separator"></div>
      <div>
        <div 
          v-for="item in groupedItems" 
          :style="{ color: item.color }" 
          :class="{ active : item.isActive }" 
          @click="moveTo(item)" >
          ❤ {{ item.count }}
          <!-- <span v-for="s in item.count">{{ s }}</span> -->
        </div>
      </div>
  </div>
</template>
<script>
import { onMounted, onUnmounted } from 'vue'
import { useSound } from '@vueuse/sound'
import spriteSfx from '../assets/sound/colors_and_digits.mp3'
import Button from './Components/Button.vue'

const useKeyboardBindings = (map) => {
  const handlePress = (ev) => {
    const handler = map[ev.key]
    if (typeof handler === 'function') {
      handler()
    }
  }
  onMounted(() => {
    window.addEventListener('keydown', handlePress)
  })
  onUnmounted(() => {
    window.removeEventListener('keydown', handlePress)
  })
}
export default {
  data() {
    return {
      rowHeight: 10,
      columnWidth: 10,
      run: false,
      spb: 1,
      round: 30,
      line: [],
      repeat: 1,
      configKonva: {
        width: 1000,
        height: 300
      },
      isDrawing: false,
      selectedColor: 'black',
      colors: [
        'red',
        'green',
        'blue',
        'white',
        'yellow',
        'pink',
        'brown'
      ]
      // configCircle: {
      //   x: 100,
      //   y: 100,
      //   radius: 70,
      //   fill: "red",
      //   stroke: "black",
      //   strokeWidth: 4
      // }
    }
  },
  methods: {
    pollData () { // second per BOOSING ^)))))))))
        let frequency = this.spb * 1000
        let isNextActive = false
        let isLast = false
        this.polling = setInterval(() => {
          if (this.run) {
            this.line.forEach((item, index, line) => {
              line.id = index
              if (isNextActive && item.isActive) {
                item.isActive = false
                if (index+1 == line.length) {
                  isLast = true
                } else {
                  line[index+1].isActive = true
                  isNextActive = false
                }
              }
              if (item.isActive) {
                // console.log(item.count*1);
                
                this.play({id: item.color})

                setTimeout(
                  () => this.play({id: item.count.toString()}), 
                  800
                )
                
                isNextActive = true
              }
            });
          }
        }, frequency)
    },
    onInputTimeInterval (e) {
      let boosing = e.target.value*1
      if (Number.isInteger(boosing) && boosing > 0) {
        clearInterval(this.polling)
        this.spb = e.target.value
        this.pollData();
      }
    },
    onInputRound (e) {
      let round = e.target.value*1
      this.round = round
    },
    onInputRepeat (e) {
      let repeat = e.target.value*1
      this.repeat = repeat
    },
    tooglePlay () {
      this.run = !this.run
    },
    reset () {
      clearInterval(this.polling)
      this.line.forEach((el,i,a) => {
        if(!i)
          el.isActive = true
        else
          el.isActive = false
      })
      this.pollData()
    },
    moveTo (item) {
      clearInterval(this.polling)
      this.line.forEach((el,i,a) => {
        if(item.id == el.id)
          el.isActive = true
        else
          el.isActive = false
      })
      this.pollData()
    },
    getConfig(item){
      // console.log(item);
      let y = Math.floor(item.id / this.round) * this.rowHeight + 50
      let x = (item.id % this.round) * this.columnWidth
      // console.log(item);
      return {
        x: x,
        y: y,
        width: this.columnWidth,
        height: this.rowHeight,
        fill: item.color,
        stroke: 'black',
      }
    },
    async fetchData( id = 0 ) {
      if(localStorage.getItem('line')){
        this.line = JSON.parse(localStorage.getItem('line'))
        console.log('INIT FROM LOCAL STORAGE');

      } else {
        console.log('INIT FROM API');
        try {
            this.data = await axios.get(`/api/get/${id}`)
              .then( (res) => {
                this.line = res.data.raports.main.content
                this.repeat = res.data.config.raport_repeat

              });
        } catch (error) {
            // Do something with the error
            console.log(error);
        }
      }
    },
    beginDrawing (e) {
      this.isDrawing = true
      this.remap(e.target.index)
      
    }, 
    keepDrawing (e) {
      if (this.isDrawing)
        this.remap(e.target.index)
    }, 
    stopDrawing () {
      this.isDrawing = false
    },
    remap (index) {
      const newLine = this.uniqueLine.reduce(
        (a,c,i) => {
        
          var clone = { ...c }
          clone.count = 1

          if((index % this.lineLength) === c.number) {
            clone.color = this.selectedColor
          }

          if(!i){
            return [clone]
          }

          if (a[a.length-1].color === clone.color) {
            a[a.length-1].count++
          } else {
            a.push(clone)
          }
          
          return a;
      },[])
      this.line = newLine
      
    },
    selectColor (color) {
      this.selectedColor = color
    }
  },
  created () {
    this.pollData()
    this.fetchData()
    window.addEventListener('beforeunload', (event) => {
      localStorage.setItem("line", JSON.stringify(this.line));
    })
  },
  components: { Button },
  setup() {
    onMounted(() => {

    })
    const { play } = useSound(spriteSfx, {
      sprite: {
        black: [700, 1000],
        blue: [2200, 1000],
        brown: [3500, 1000],
        green: [5400, 1000],
        orange: [7100, 1000],
        pink: [8600, 1000],
        purple: [10000, 1000],
        red: [11600, 1000],
        white: [13300, 1000],
        yellow: [14700, 1000],
        '0': [16000, 800],
        '1': [17200, 800],
        '2': [18000, 800],
        '3': [18700, 800],
        '4': [19700, 800],
        '5': [20700, 800],
        '6': [21600, 800],
        '7': [22500, 800],
        '8': [23400, 700],
        '9': [24200, 800],
      },
      playbackRate: 1.1,
      // interrupt: false,
    })
    // useKeyboardBindings({
    //   0: () => play({ id: '0' }),
    //   1: () => play({ id: '1' }),
    //   2: () => play({ id: '2' }),
    //   3: () => play({ id: '3' }),
    //   4: () => play({ id: '4' }),
    //   5: () => play({ id: '5' }),
    //   6: () => play({ id: '6' }),
    //   7: () => play({ id: '7' }),
    //   8: () => play({ id: '8' }),
    //   9: () => play({ id: '9' }),
    // })

    return {
      play,
    }
  },
  computed: {
    groupedItems() {
      return this.line
      console.log(this.chunk(this.line, this.round));
      // console.log(this.line[0].color);
      return this.line.slice().reverse()
    },
    bitmap() {
      var count = 0;
      // console.log(this.line);
      // for(let i=0; i > this.)
      const raport = this.line.reduce(
        (accumulator, currentValue) => {
          for(let i = 1; i <= currentValue.count; i++) {
            let clone = { 
              id: currentValue.id,
              color: currentValue.color
            }
            clone.number = count
            count++
            accumulator.push(clone);
          }
          return accumulator
        },
        [],
      );

      var bitmap = []

      for(let i = 0; i < raport.length * this.repeat; i++) {
        let current = raport[ i % raport.length ];
        let clone = { ...current }
        clone.id = i
        bitmap.push(clone)
      }
      // let bitmap = Array(this.repeat).fill(raport).flat()
      // console.log(bitmap);
      
      return bitmap
    },
    lineLength () {
      return this.line.reduce( (a,c) => a+c.count, 0)
    },
    uniqueLine() {
      return this.bitmap.slice(0, this.lineLength)
    }
  }
}
</script>

<style>
  .active{
    border: 2px solid goldenrod;
  }
  .play {
    background-color: green;
    color: red;
  }
  .color-selector {
    width: 20px;
    height: 20px;
    margin: 5px;
    float: left;
  }
  .separator {
    clear: both;
  }
</style>