<template >
  <div class="pa-3" @click="setControlFocus()">
    <input
      type="text"
      ref="controls"
      v-on:keydown.up="move(0, -1)"
      v-on:keydown.down="move(0, 1)"
      v-on:keydown.left="move(-1, 0)"
      v-on:keydown.right="move(1, 0)"
      style="width:0;"
    />

    <div class="d-flex">
      <div class="store" :class="{ edit: editMode }" :style="{ width: matrixW, height: matrixH }" style="position:relative;">
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

        <div class="d-flex flex-row" v-for="(row, r) in store.cells" :key="r + 1000">
          <div class="cell" v-for="(cell, c) in row" :key="c + 2000" @click="selCell(r, c)">
            <template v-if="cell.in === 'S'">
              <img src="/autostore/power.png" class="station" />
            </template>
          </div>
        </div>
      </div>
      <div class="commands">
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

        <div>
          <v-checkbox v-model="editMode" label="Edit level"></v-checkbox>
        </div>
        <div v-if="editMode">
          <hr />
          <v-btn height="100" text><v-img src="/autostore/roboth.png" contain height="80"></v-img></v-btn>
          <v-btn height="100" text><v-img src="/autostore/robotv.png" contain height="80"></v-img></v-btn>
          <v-btn height="100" text><v-img src="/autostore/power.png" contain height="80"></v-img></v-btn>
          <v-btn fab depressed class="ml-3" color=""><v-icon>mdi-checkbox-intermediate</v-icon></v-btn>
          <v-btn height="100" text class="ml-3" color="red "><v-icon large>mdi-delete-forever</v-icon></v-btn>
        </div>
      </div>
    </div>
    <!-- .flex -->

    <p></p>
    <v-btn color="secondary" @click="newGame">New game</v-btn>
  </div>
</template>

<script>
const cellWidth = 70
const cellHeight = 70
const padding = 6
const publicPath = process.env.BASE_URL

class Cell {
  constructor(elt = '') {
    // ''=empty , 'P'=player, 'R'=robot, 'C'=robot Charging, 'S'=power Station
    this.in = elt
  }
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
  constructor(nbRows, nbCols, nbRobots) {
    this.cells = []
    this.nbRows = nbRows
    this.nbCols = nbCols
    this.nbRobots = nbRobots
    for (let r = 0; r < nbRows; r++) {
      let row = []
      for (let c = 0; c < nbCols; c++) {
        row.push(new Cell(''))
      }
      this.cells.push(row)
    }
    this.player = new Player(nbRows - 1, 0)
    this.robots = []
  }
  empty() {
    this.cells.forEach(row => {
      row.forEach(cell => {
        cell.in = ''
      })
    })
  }
  initStore() {
    this.empty()
    this.robots = []
    /**random robots positions */
    for (let n = 0; n < this.nbRobots; n++) {
      for (let t = 0; t < 20; t++) {
        //20 attempts max
        let r = getRandomInt(nbRows - 2) + 1
        let c = getRandomInt(nbCols - 2) + 1
        if (this.cells[r][c].in === '' && !this.blockingTest(r, c, ['R', 'C'])) {
          // ok empty
          this.cells[r][c].in = 'R'
          let orientation = getRandomInt(2)
          this.robots.push(new Robot(r, c, orientation === 1 ? 'H' : 'V'))
          break
        }
      }
      /**random charging stations */
      for (let t = 0; t < 20; t++) {
        //20 attempts max
        let r = getRandomInt(nbRows)
        let c = getRandomInt(nbCols)
        if (this.cells[r][c].in === '' && !this.blockingTest(r, c, ['R', 'C', 'S'])) {
          // ok empty
          this.cells[r][c].in = 'S'
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
    if (row > 0) up = this.cells[row - 1][col].in
    let down = ''
    if (row + 1 < this.nbRows) down = this.cells[row + 1][col].in
    let left = ''
    if (col > 0) left = this.cells[row][col - 1].in
    let right = ''
    if (col + 1 < this.nbCols) right = this.cells[row][col + 1].in

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
    if (indRobot === -1) return false
    let robot = this.robots[indRobot]

    if (hd !== 0 && robot.orientation === 'V') return false
    if (vd !== 0 && robot.orientation === 'H') return false

    if (this.cells[nRow][nCol].in === '') {
      this.cells[row][col].in = ''
      this.cells[nRow][nCol].in = 'R'
      robot.moveTo(nRow, nCol)
      return true
    }
    if (this.cells[nRow][nCol].in === 'S') {
      this.cells[row][col].in = ''
      this.cells[nRow][nCol].in = 'C'
      robot.moveTo(nRow, nCol)
      setTimeout(() => {
        robot.setCharging()
      }, 500)

      return true
    }
    return false
  }
  newPlayerPos(row, col) {
    this.cells[this.player.row][this.player.col].in = ''
    this.player.moveTo(row, col)
    this.cells[row][col].in = 'P'
  }
  movePlayer(hd, vd) {
    let nCol = this.player.col + hd
    let nRow = this.player.row + vd
    if (nCol < 0 || nCol >= this.nbCols || nRow < 0 || nRow >= this.nbRows) return

    if (this.cells[nRow][nCol].in === '') {
      this.newPlayerPos(nRow, nCol)
    }
    if (this.cells[nRow][nCol].in === 'R') {
      if (this.moveRobot(hd, vd, nRow, nCol)) {
        this.newPlayerPos(nRow, nCol)
      }
    }
  }
}
const nbRobots = 7
const nbRows = 8
const nbCols = 8
function getRandomInt(max) {
  return Math.floor(Math.random() * Math.floor(max))
}

export default {
  name: 'Game',
  data: () => ({
    store: new Store(nbRows, nbCols, nbRobots),
    matrixH: nbRows * 70 + 32 + 'px',
    matrixW: nbCols * 70 + 32 + 'px',
    editMode: false,
    editRow: null,
    editCol: null
  }),
  methods: {
    move(hd, vd) {
      this.store.movePlayer(hd, vd)
    },
    setControlFocus() {
      this.$refs.controls.focus()
    },
    newGame() {
      this.store.initStore()
    },
    selCell(r, c) {
      this.editRow = r
      this.editCol = c
      if (this.editMode) {
        let cell = this.store.cells[r][c].in
        if (cell === 'R') {
          let indRobot = this.store.robots.findIndex(R => {
            return R.col === c && R.row === r
          })
          this.store.robots.splice(indRobot, 1)
        }
        this.store.cells[r][c].in = ''
      }
    }
  },
  created() {
    this.newGame()
  },
  mounted() {
    this.setControlFocus()
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
