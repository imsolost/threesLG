<template>
  <div id="app">
    <div class="grid">
      <div v-for="n in 6" class="row">
        <div v-for="n in 6" class="cell"></div>
      </div>

      <div v-for="tile in tiles"
            v-bind:id="tile.id"
            v-bind:style="{top: `${ tile.y * 60}px`, left: `${ tile.x * 60}px`}"
            class="tile">
        {{tile.number}}
      </div>

      <div v-show="score" class="game-over-shroud">
        <div class="score">
          Your score is {{score}}
        </div>
        <div class="game-over-text">
          Game Over
        </div>
        <div class="new-game">
          New Game
        </div>
      </div>
    </div>
  </div>
</template>

<script>

const transitionInterval = 0

export default {
  name: 'app',
  created: function () {
    this.newGame()
    window.addEventListener('keydown', this.move)
  },
  data: function () {
    return {
      tiles: [],
      score: 0
    }
  },
  methods: {
    newGame() {
      this.initializeTiles()
      this.score = 0
    },

    initializeTiles() {
      this.tiles = []

      let tileNumbers = [1, 1, 2, 2, 3, 3]

      for(let i = 0; i < 3; i++) {
        tileNumbers.push(Math.floor(Math.random() * 3 + 1))
      }

      tileNumbers.forEach(number => {
        let x, y

        do {
          x = Math.floor(Math.random() * 4 + 1)
          y = Math.floor(Math.random() * 4 + 1)
        } while (this.tiles.find(tile => tile.x === x && tile.y === y))

        this.tiles.push({x, y, number})
      })
    },

    gameOverCheck () {
      const availableCols = []
      const availableRows = []

      for (let colNum = 1; colNum <= 4; colNum++) {
        if (this.willThereBeSpaceInCol(colNum)) {
          availableCols.push(colNum)
        }
      }

      for (let rowNum = 1; rowNum <= 4; rowNum++) {
        if (this.willThereBeSpaceInRow(rowNum)) {
          availableRows.push(rowNum)
        }
      }

      if (availableRows.length !== 0 || availableCols.length !== 0) return
      else if (availableRows.length === 0 && availableCols.length === 0) {
        // setTimeout(() => alert('Game Over'), 200)

        this.score = this.calculateScore()
      }
    },

    calculateScore () {
      let score = 0
      this.tiles.forEach(tile => {
        if (tile.number > 2) {
          score += 3 ** (Math.log2(tile.number / 3) + 1)
        }
      })
      return score
    },

    insertTile (side) {
      let targetRow, targetCol
      switch (side) {
        case 'left':
          targetRow = this.randomAvailbleRow()
          if (targetRow) {
            this.tiles.push({x: 0, y: targetRow, number: this.pickRandomTileNumber()})
          }
          break;
        case 'right':
          targetRow = this.randomAvailbleRow()
          if (targetRow) {
            this.tiles.push({x: 5, y: targetRow, number: this.pickRandomTileNumber()})
          }
          break;
        case 'top':
          targetCol = this.randomAvailbleCol()
          if (targetCol) {
            this.tiles.push({x: targetCol, y: 0, number: this.pickRandomTileNumber()})
          }
          break;
        case 'bottom':
          targetCol = this.randomAvailbleCol()
          if (targetCol) {
            this.tiles.push({x: targetCol, y: 5, number: this.pickRandomTileNumber()})
          }
          break;
        default:

      }
    },

    pickRandomTileNumber() {
      const insertableTileNumbers = [1,1,2,2,3]
      const randomIndex = Math.floor(Math.random() * insertableTileNumbers.length)

      return insertableTileNumbers[randomIndex]
    },

    randomAvailbleRow() {
      const availableRows = []
      for (let rowNum = 1; rowNum <= 4; rowNum++) {
        if (this.willThereBeSpaceInRow(rowNum)) {
          availableRows.push(rowNum)
        }
      }

      if (availableRows.length === 0) return undefined

      const randomIndex = Math.floor(Math.random() * availableRows.length)

      return availableRows[randomIndex]
    },

    willThereBeSpaceInRow(rowNum) {
      const rowTiles = []
      let isEmptySpace = false

      for (let colNum = 1; colNum <= 4; colNum++) {
        rowTiles[colNum] = this.getTileByPosition(colNum, rowNum)
        if (!rowTiles[colNum]) isEmptySpace = true
      }

      if (isEmptySpace) return true

      for (let colNum = 1; colNum <= 3; colNum++) {
        const num1 = rowTiles[colNum].number
        const num2 = rowTiles[colNum + 1].number

        if ((num1 === num2 && num1 > 2) || (num1 + num2 === 3)) {
          return true
        }
      }

      return false
    },

    randomAvailbleCol() {
      const availableCols = []
      for (let colNum = 1; colNum <= 4; colNum++) {
        if (this.willThereBeSpaceInCol(colNum)) {
          availableCols.push(colNum)
        }
      }

      if (availableCols.length === 0) return undefined

      const randomIndex = Math.floor(Math.random() * availableCols.length)

      return availableCols[randomIndex]
    },

    willThereBeSpaceInCol(colNum) {
      const colTiles = []
      let isEmptySpace = false

      for (let rowNum = 1; rowNum <= 4; rowNum++) {
        colTiles[rowNum] = this.getTileByPosition(colNum, rowNum)
        if (!colTiles[rowNum]) isEmptySpace = true
      }

      if (isEmptySpace) return true

      for (let rowNum = 1; rowNum <= 3; rowNum++) {
        const num1 = colTiles[rowNum].number
        const num2 = colTiles[rowNum + 1].number

        if ((num1 === num2 && num1 > 2) || (num1 + num2 === 3)) {
          return true
        }
      }

      return false
    },

    move: function (event) {
      console.log('event', event)
      if (event.code === 'ArrowRight') {
        event.preventDefault()

        this.insertTile('left')

        setTimeout( () => {
          this.moveColumn(3, 'right')
          this.moveColumn(2, 'right')
          this.moveColumn(1, 'right')
          this.moveColumn(0, 'right')
          this.gameOverCheck()
        }, 5)

      } else if (event.code === 'ArrowLeft') {
        event.preventDefault()

        this.insertTile('right')
        setTimeout( () => {
          this.moveColumn(2, 'left')
          this.moveColumn(3, 'left')
          this.moveColumn(4, 'left')
          this.moveColumn(5, 'left')
          this.gameOverCheck()
        }, 5)

      } else if (event.code === 'ArrowUp') {
        event.preventDefault()

        this.insertTile('bottom')
        setTimeout( () => {
          this.moveRow(2, 'up')
          this.moveRow(3, 'up')
          this.moveRow(4, 'up')
          this.moveRow(5, 'up')
          this.gameOverCheck()
        }, 5)

      } else if (event.code === 'ArrowDown') {
        event.preventDefault()

        this.insertTile('top')
        setTimeout( () => {
          this.moveRow(3, 'down')
          this.moveRow(2, 'down')
          this.moveRow(1, 'down')
          this.moveRow(0, 'down')
          this.gameOverCheck()
        }, 5)
      } else if (event.code === 'Space') {
        event.preventDefault()

        this.newGame()
      }
    },

    combineIfPossible (movingTile, targetTile) {
      const num1 = movingTile.number
      const num2 = targetTile.number

      if ((num1 === num2 && num1 > 2) || (num1 + num2 === 3)) {
        targetTile.number = num1 + num2
        // setTimeout(() =>
          this.tiles.splice(this.tiles.indexOf(movingTile), 1)
        //   transitionInterval
        // )
      }
    },

    getTileByPosition (x, y) {
      return this.tiles.find(tile => tile.x === x && tile.y === y)
    },

    moveColumn: function (colNum, direction) {
      const increment = direction === 'right' ? 1 : -1

      const tilesInCol = this.tiles.filter(tile => tile.x === colNum)

      tilesInCol.forEach(tile => {
        const targetTile = this.getTileByPosition(tile.x + increment, tile.y)
        if (!targetTile) {
          tile.x = tile.x + increment
        } else {
          this.combineIfPossible(tile, targetTile)
        }
      })
    },

    moveRow: function (rowNum, direction) {
      const increment = direction === 'down' ? 1 : -1

      const tilesInRow = this.tiles.filter(tile => tile.y === rowNum)

      tilesInRow.forEach(tile => {
        const targetTile = this.getTileByPosition(tile.x , tile.y + increment)
        if (!targetTile) {
          tile.y = tile.y + increment
        } else {
          this.combineIfPossible(tile, targetTile)
        }
      })
    }
  }
}

</script>

<style>
/** {
  filter: invert(100%);
}*/

.row {
  display: flex;
}

.cell {
  width: 50px;
  height: 50px;
  border: 5px red solid;
}

.grid {
  position: relative;
  background-color: white;
}

.tile {
  width: 50px;
  height: 50px;
  border: 5px blue solid;
  border-radius: 10px;
  box-shadow: inset 0 0 10px black;
  position: absolute;
  text-align: center;
  font-size: 25px;
  font-style: bold;
  line-height: 50px;
  background-color: grey;
  transition: 0ms;
}

.game-over-shroud {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  background-color: rgba(127, 127, 127, 0.7);
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
}
</style>
