<template>
  <div id="app">

    <div class="next-tile-cell">
      <div v-bind:class="baseTileNumbers.slice(-1)[0] === 1
        ? 'tile-one tile next-tile'
          : baseTileNumbers.slice(-1)[0] === 2
          ? 'tile-two tile next-tile'
            : 'tile next-tile' ">
        <!-- {{baseTileNumbers.slice(-1)[0]}} -->
      </div>
      <div>
        Next
      </div>
    </div>


    <div class="tile-grid">
      <div v-for="n in 4" class="row">
        <div v-for="n in 4" class="cell"></div>
      </div>

      <div v-for="tile in tiles"
            :key="tile.id"
            v-bind:style="{
              top: `${ (tile.y - 1) * (86.90169943749474 + 10) + 10}px`,
              left: `${ (tile.x - 1) * (56 + 10) + 13}px`
              }"
            v-bind:class="
              tile.number === 1 ? 'tile-one tile' :
              tile.number === 2 ? 'tile-two tile' : 'tile' ">
        {{tile.number}}
      </div>
    </div>

    <div v-on:click="newGame" class="new-game-button">
      New Game
    </div>

    <div v-show="score" class="game-over-shroud">

      <div class="game-over-text">
        Game Over
      </div>
      <div class="score">
        Score: {{score}}
      </div>

      <div v-on:click="newGame" class="new-game-shroud-button">
        Try Again
      </div>
    </div>

  </div>
</template>

<script>

const transitionInterval = 250

export default {
  name: 'app',
  created: function () {
    this.newGame()
    window.addEventListener('keydown', this.move)
  },
  data: function () {
    return {
      tiles: [],
      score: 0,
      locked: false,
      baseTileNumbers: [],
      bigTileNumbers: []
    }
  },
  methods: {
    newGame() {
      this.initializeTiles()
      this.score = 0
    },

    initializeTiles() {
      this.tiles = []

      this.resetBaseTileNumbers()

      const tileNumbers = this.baseTileNumbers.splice(3, 9)

      tileNumbers.forEach(number => {
        let x, y

        do {
          x = Math.floor(Math.random() * 4 + 1)
          y = Math.floor(Math.random() * 4 + 1)
        } while (this.tiles.find(tile => tile.x === x && tile.y === y))

        this.tiles.push({x, y, number, id: Math.random()})
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
            this.tiles.push({x: 0, y: targetRow, number: this.pickRandomTileNumber(), id: Math.random()})
          }
          break;
        case 'right':
          targetRow = this.randomAvailbleRow()
          if (targetRow) {
            this.tiles.push({x: 5, y: targetRow, number: this.pickRandomTileNumber(), id: Math.random()})
          }
          break;
        case 'top':
          targetCol = this.randomAvailbleCol()
          if (targetCol) {
            this.tiles.push({x: targetCol, y: 0, number: this.pickRandomTileNumber(), id: Math.random()})
          }
          break;
        case 'bottom':
          targetCol = this.randomAvailbleCol()
          if (targetCol) {
            this.tiles.push({x: targetCol, y: 5, number: this.pickRandomTileNumber(), id: Math.random()})
          }
          break;
        default:

      }
    },

    highestNumber() {
      return this.tiles.reduce((accumulator, tile) => {
        return tile.number > accumulator ? tile.number : accumulator
      }, 0)
    },

    pickRandomTileNumber() {
      let randomTileNumber = this.baseTileNumbers.pop()

      if (this.baseTileNumbers.length === 0) {
        this.resetBaseTileNumbers()
      }

      const largestNumber = this.highestNumber()

      if (largestNumber >= 48) {

        for (let number = largestNumber / 8; number >= 6 ; number /= 2) {
          this.bigTileNumbers.push(number)
        }

        if (Math.random() < 1 / 21) {
          const randomIndex = Math.floor(Math.random() * this.bigTileNumbers.length)

          this.baseTileNumbers.push(this.bigTileNumbers[randomIndex])
        }

      }

      return randomTileNumber
    },

    resetBaseTileNumbers() {
      const unsortedNumbers = [1, 1, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3]
      let randomIndex, randomNumber

      this.baseTileNumbers = []

      while (unsortedNumbers.length > 0) {
        randomIndex = Math.floor(Math.random() * unsortedNumbers.length)
        randomNumber = unsortedNumbers.splice(randomIndex, 1)[0]
        this.baseTileNumbers.push(randomNumber)
      }
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
      if (this.locked) return

      this.locked = true
      setTimeout(() => this.locked = false, transitionInterval)

      if (event.code === 'ArrowRight') {
        event.preventDefault()

        this.insertTile('left')

        setTimeout( () => {
          this.moveColumn(3, 'right')
          this.moveColumn(2, 'right')
          this.moveColumn(1, 'right')
          this.moveColumn(0, 'right')
          setTimeout(() => this.gameOverCheck(), transitionInterval + 5)
        }, 5)

      } else if (event.code === 'ArrowLeft') {
        event.preventDefault()

        this.insertTile('right')
        setTimeout( () => {
          this.moveColumn(2, 'left')
          this.moveColumn(3, 'left')
          this.moveColumn(4, 'left')
          this.moveColumn(5, 'left')
          setTimeout(() => this.gameOverCheck(), transitionInterval + 5)
        }, 5)

      } else if (event.code === 'ArrowUp') {
        event.preventDefault()

        this.insertTile('bottom')
        setTimeout( () => {
          this.moveRow(2, 'up')
          this.moveRow(3, 'up')
          this.moveRow(4, 'up')
          this.moveRow(5, 'up')
          setTimeout(() => this.gameOverCheck(), transitionInterval + 5)
        }, 5)

      } else if (event.code === 'ArrowDown') {
        event.preventDefault()

        this.insertTile('top')
        setTimeout( () => {
          this.moveRow(3, 'down')
          this.moveRow(2, 'down')
          this.moveRow(1, 'down')
          this.moveRow(0, 'down')
          setTimeout(() => this.gameOverCheck(), transitionInterval + 5)
        }, 5)
      } else if (event.code === 'Space') {
        event.preventDefault()

        this.newGame()
      }
    },

    combineIfPossible (movingTile, targetTile, increment, direction) {
      const num1 = movingTile.number
      const num2 = targetTile.number

      if ((num1 === num2 && num1 > 2) || (num1 + num2 === 3)) {

        if (direction === 'horizontal') {
          movingTile.x = movingTile.x + increment
        } else if (direction === 'vertical') {
          movingTile.y = movingTile.y + increment
        }

        setTimeout(() => {
            this.tiles.splice(this.tiles.indexOf(movingTile), 1)
            targetTile.number = num1 + num2
          },
          transitionInterval
        )
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
          this.combineIfPossible(tile, targetTile, increment, 'horizontal')
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
          this.combineIfPossible(tile, targetTile, increment, 'vertical')
        }
      })
    }
  }
}

</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Fira+Sans+Condensed');

#app {
  font-family: 'Fira Sans Condensed', sans-serif;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.row {
  display: flex;
}

.cell {
  width: 56px;
  height: 86.90169943749474px;
  background-color: #bbd9d9;
  border-radius: 5px;
  margin: 5px;
}

.tile-grid {
  position: relative;
  margin-bottom: 21px;
  background-color: #cfe7e0;
  border-radius: 15px;
  padding: 5px;
  border-bottom: 7px #bbd9d9 solid;
}

.tile {
  width: 50px;
  height: 75.90169943749474px;
  border-bottom: 5px #ffcc66 solid;
  border-radius: 7px;
  position: absolute;
  text-align: center;
  font-size: 35px;
  font-style: bold;
  line-height: 80.90169943749474px;
  background-color: white;
  transition: 250ms;
  font-family: 'Fira Sans Condensed', sans-serif;
}

.next-tile {
  height: 36.45px;
  width: 25px;
  font-size: 18px;
  line-height: 37.95px;
  position: relative;
  margin-bottom: 5px;
  border-bottom-width: 4px;
  border-radius: 4px;
}

.next-tile-cell {
  padding: 5px;
  background-color: #cfe7e0;
  border-radius: 5px;
  margin-top: 25px;
  margin-bottom: 25px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.game-over-shroud {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #ddd;
  font-size: 30px;
  line-height: 60px;
}

.tile-one {
  background-color: #72caf2;
  color: white;
  border-color: #6ba6da;
}

.tile-two {
  background-color: #f16780;
  color: white;
  border-color: #cd537c;
}

.new-game-button {
  text-align: center;
  font-size: 20px;
  background-color: #777e8c;
  border-bottom: 5px #494766 solid;
  border-radius: 5px;
  color: white;
  margin-bottom: 25px;
  padding: 4px 15px;
}

.new-game-button:hover, .new-game-shroud-button {
  cursor: pointer;
}

.new-game-shroud-button {
  text-align: center;
  background-color: hsl(220, 8%, 30%);
  border-radius: 5px;
  border-bottom: 5px hsl(220, 8%, 12%) solid;
  position: absolute;
  bottom: 22px;
  font-size: 20px;
  line-height: 30px;
  padding: 4px 25px;
}

</style>
