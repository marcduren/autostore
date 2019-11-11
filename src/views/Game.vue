<template >
  <div class="pa-3" @click="setControlFocus()">
    <div class="white--text font-weight-bold">{{ store.nbMoves }} Movement<span v-if="store.nbMoves > 1">s</span></div>
    <div class="d-flex flex-wrap">
      <div class="store" :class="{ edit: editMode }" :style="{ width: matrixW, height: matrixH }" style="position:relative;">
        <!-- robots -->
        <img
          v-for="(R, ind) in store.robots"
          :src="R.imgSrc"
          class="iRobot"
          :key="ind"
          :style="{ left: R.x + 'px', top: R.y + 'px' }"
        />
        <!-- player -->
        <img
          v-if="!editMode"
          :src="store.player.imgSrc"
          class="iRobot"
          :style="{ left: store.player.x + 'px', top: store.player.y + 'px' }"
        />
        <!-- edit pos -->
        <v-icon
          v-if="editMode"
          style="position:absolute"
          :style="{ left: editCol * 70 + 30 + 'px', top: editRow * 70 + 30 + 'px' }"
          >mdi-map-marker</v-icon
        >
        <!-- matrix -->
        <div class="d-flex flex-row" v-for="(row, r) in store.cells" :key="r + 1000">
          <div class="cell" v-for="(cell, c) in row" :key="c + 2000" @click="selCell(r, c)">
            <template v-if="cell === 'S'">
              <img src="/autostore/power.png" class="station" />
            </template>
          </div>
        </div>
        <!-- if completed -->
        <div
          v-if="store.levelCompleted"
          class="yellow--text font-weight-bold display-2"
          style="position:absolute;top:30%;left:10%;z-index:4;"
        >
          COMPLETED
        </div>
      </div>
      <div class="commands">
        <v-select label="Playing level" v-model="level" :items="levels" @input="newLevel" style="max-width:100px"></v-select>
        <v-btn @click="newLevel">Replay</v-btn>
        <!--v-checkbox v-model="editMode" label="Edit level"></v-checkbox--><!-- in development-->
        <div v-if="editMode">
          <v-btn color="secondary" @click="random">Random</v-btn>
          <hr />
          <v-btn height="100" text><v-img src="/autostore/roboth.png" contain height="80"></v-img></v-btn>
          <v-btn height="100" text><v-img src="/autostore/robotv.png" contain height="80"></v-img></v-btn>
          <v-btn height="100" text><v-img src="/autostore/power.png" contain height="80"></v-img></v-btn>
        </div>
      </div>
    </div>
    <table>
      <tr>
        <td></td>
        <td>
          <v-btn color="primary" @click="move(0, -1)"><v-icon>mdi-arrow-up-bold-outline</v-icon></v-btn>
        </td>
        <td></td>
      </tr>
      <tr>
        <td>
          <v-btn color="primary" @click="move(-1, 0)"><v-icon>mdi-arrow-left-bold-outline</v-icon></v-btn>
        </td>
        <td></td>
        <td>
          <v-btn color="primary" @click="move(1, 0)"><v-icon>mdi-arrow-right-bold-outline</v-icon></v-btn>
        </td>
      </tr>
      <tr>
        <td></td>
        <td>
          <v-btn color="primary" @click="move(0, 1)"><v-icon>mdi-arrow-down-bold-outline</v-icon></v-btn>
        </td>
        <td></td>
      </tr>
    </table>

    <!-- .flex -->

    <p></p>
  </div>
</template>

<script>
const cellWidth = 70
const cellHeight = 70
const padding = 6
const publicPath = process.env.BASE_URL
import levelsdef from '../plugins/levels.json'

function getRandomInt(max) {
  return Math.floor(Math.random() * Math.floor(max))
}

class Robot {
  constructor(row, col, orientation) {
    this.row = row
    this.col = col
    this.orientation = orientation // H or V
    this.charging = false
    if (orientation === 'H') {
      this.imgSrc = publicPath + 'roboth.png'
    } else {
      this.imgSrc = publicPath + 'robotv.png'
    }
    this.img = this.x = col * cellWidth + padding
    this.y = row * cellHeight + padding
  }
  moveTo(row, col) {
    this.row = row
    this.col = col
    this.x = col * cellWidth + padding
    this.y = row * cellHeight + padding
  }
  setCharging() {
    this.charging = true
    if (this.orientation === 'H') {
      this.imgSrc = publicPath + 'robothc.png'
    } else {
      this.imgSrc = publicPath + 'robotvc.png'
    }
  }
}
class Player {
  constructor(row, col) {
    this.row = row
    this.col = col
    this.imgSrc = publicPath + 'player.png'
    this.x = col * cellWidth + padding
    this.y = row * cellHeight + padding
  }
  moveTo(row, col) {
    this.row = row
    this.col = col
    this.x = col * cellWidth + padding
    this.y = row * cellHeight + padding
  }
}

class Store {
  constructor(level) {
    let defs = levelsdef[level - 1]
    this.level = level
    this.cells = [] // ''=empty , 'P'=Player, Robot, 'C'=robot Charging, 'S'=charging Station
    this.nbRows = defs.nbRows
    this.nbCols = defs.nbCols
    this.nbRobots = defs.nbRobots
    this.robots = []
    for (let r = 0; r < this.nbRows; r++) {
      let row = []
      for (let c = 0; c < this.nbCols; c++) {
        let t = defs.cells[r][c]
        if (t == 'RH') {
          t = 'R'
          this.robots.push(new Robot(r, c, 'H'))
        }
        if (t == 'RV') {
          t = 'R'
          this.robots.push(new Robot(r, c, 'V'))
        }
        row.push(t)
      }
      this.cells.push(row)
    }
    this.player = new Player(defs.nbRows - 1, 0)
    this.nbMoves = 0
    this.levelCompleted = false
  }
  empty() {
    this.cells.forEach(row => {
      for (let c = 0; c < this.nbCols; c++) {
        row[c] = ''
      }
    })
  }
  randomStore() {
    this.empty()
    this.robots = []
    /**random robots positions */
    for (let n = 0; n < this.nbRobots; n++) {
      for (let t = 0; t < 20; t++) {
        //20 attempts max
        let r = getRandomInt(this.nbRows - 2) + 1
        let c = getRandomInt(this.nbCols - 2) + 1
        if (this.cells[r][c] === '' /*&& !this.blockingTest(r, c, ['R', 'C'])*/) {
          // ok empty
          this.cells[r][c] = 'R'
          let orientation = getRandomInt(2)
          this.robots.push(new Robot(r, c, orientation === 1 ? 'H' : 'V'))
          break
        }
      }
      /**random charging stations */
      for (let t = 0; t < 20; t++) {
        //20 attempts max
        let r = getRandomInt(this.nbRows)
        let c = getRandomInt(this.nbCols)
        if (this.cells[r][c] === '' /*&& !this.blockingTest(r, c, ['R', 'C', 'S'])*/) {
          // ok empty
          this.cells[r][c] = 'S'
          break
        }
      }
    }

    //player
    this.newPlayerPos(this.nbRows - 1, 0)
  }
  blockingTest(row, col, types) {
    /** row,col: number, types: array<string> */
    // return true if cell(row,col) is blocking (not possible) by adjacents
    let up = ''
    if (row > 0) up = this.cells[row - 1][col]
    let down = ''
    if (row + 1 < this.nbRows) down = this.cells[row + 1][col]
    let left = ''
    if (col > 0) left = this.cells[row][col - 1]
    let right = ''
    if (col + 1 < this.nbCols) right = this.cells[row][col + 1]

    if (types.includes(up) || types.includes(down) || types.includes(left) || types.includes(right)) return true
    return false
  }
  moveRobot(hd, vd, row, col) {
    /**move robot at row,col */
    let nCol = col + hd
    let nRow = row + vd
    if (nCol < 0 || nCol >= this.nbCols || nRow < 0 || nRow >= this.nbRows) return false

    let indRobot = this.robots.findIndex(R => {
      return R.col === col && R.row === row
    })
    if (indRobot === -1) return false // not possible! can't find robot as row,col
    let robot = this.robots[indRobot]

    if (hd !== 0 && robot.orientation === 'V') return false // can't move horizontaly
    if (vd !== 0 && robot.orientation === 'H') return false // can't move verticaly

    if (this.cells[nRow][nCol] === '') {
      // ok cell is empty
      this.cells[row][col] = ''
      this.cells[nRow][nCol] = 'R'
      robot.moveTo(nRow, nCol)
      return true
    }
    if (this.cells[nRow][nCol] === 'S') {
      // ok cell is a Charging station
      this.cells[row][col] = ''
      this.cells[nRow][nCol] = 'C'
      robot.moveTo(nRow, nCol)
      let nbOK = 0
      this.robots.forEach(R => {
        if (R.charging) nbOK++
      })
      console.log(nbOK)
      if (nbOK + 1 == this.nbRobots) this.levelCompleted = true

      setTimeout(() => {
        robot.setCharging()
      }, 500)
      /** test level completed */
      return true
    }
    return false
  }
  newPlayerPos(row, col) {
    this.cells[this.player.row][this.player.col] = ''
    this.player.moveTo(row, col)
    this.cells[row][col] = 'P'
    this.nbMoves++
  }
  movePlayer(hd, vd) {
    if (this.levelCompleted) return
    let nCol = this.player.col + hd
    let nRow = this.player.row + vd
    if (nCol < 0 || nCol >= this.nbCols || nRow < 0 || nRow >= this.nbRows) return

    if (this.cells[nRow][nCol] === '') {
      this.newPlayerPos(nRow, nCol)
    }
    if (this.cells[nRow][nCol] === 'R') {
      if (this.moveRobot(hd, vd, nRow, nCol)) {
        this.newPlayerPos(nRow, nCol)
      }
    }
  }
}

export default {
  name: 'Game',
  data: () => ({
    level: 1,
    levels: levelsdef.map(L => L.level),
    store: new Store(1),
    editMode: false,
    editRow: null,
    editCol: null
  }),
  computed: {
    matrixH: function() {
      return levelsdef[this.level - 1].nbRows * 70 + 32 + 'px'
    },
    matrixW: function() {
      return levelsdef[this.level - 1].nbCols * 70 + 32 + 'px'
    }
  },
  methods: {
    move(hd, vd) {
      this.store.movePlayer(hd, vd)
    },
    setControlFocus() {
      //this.$refs.controls.focus()
    },
    random() {
      this.store.randomStore()
    },
    newLevel() {
      this.store = new Store(this.level)
    },
    selCell(r, c) {
      this.editRow = r
      this.editCol = c
      if (this.editMode) {
        let cell = this.store.cells[r][c]
        if (cell === 'R') {
          let indRobot = this.store.robots.findIndex(R => {
            return R.col === c && R.row === r
          })
          this.store.robots.splice(indRobot, 1)
        }
        let tb = this.store.cells.slice()
        tb[r][c] = ''
        this.store.cells = tb //vuejs Reactivity
      }
    }
  },
  created() {},
  mounted() {
    window.addEventListener('keydown', e => {
      if (e.keyCode == 37) this.move(-1, 0)
      if (e.keyCode == 39) this.move(1, 0)
      if (e.keyCode == 38) this.move(0, -1)
      if (e.keyCode == 40) this.move(0, 1)
      console.log(e.keyCode)
    })
  }
}
</script>
<style>
.store {
  padding: 6px;
}
.cell {
  background-color: #555;
  color: black;
  width: 70px;
  height: 70px;
  border: 2px solid #aaf;
}
.edit .cell:hover {
  background-color: #777;
  cursor: pointer;
}
.edit img {
  width: 45px;
}
.iRobot {
  position: absolute;
  transition: all 0.4s;
  z-index: 2;
}
.edit .iRobot {
  transition: none 0s;
}
.robot,
.player {
  min-width: 70px;
  min-height: 70px;
  margin-top: -2px;
}

.station {
  margin-left: -1px;
  margin-top: -1px;
}
</style>
