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
    </div>
  </div>
</template>

<script>

const transitionInterval = 0

export default {
  name: 'app',
  created: function () {
    window.addEventListener('keydown', this.move)
  },
  data: function () {
    return {
      tiles: [
               {x: 1, y: 1, number: 2},
               {x: 2, y: 2, number: 2},
               {x: 3, y: 3, number: 3},
               {x: 4, y: 4, number: 2}
             ]
    }
  },
  methods: {
    insertTile (side) {
      switch (side) {
        case 'left':
          this.tiles.push({x: 0, y: 3, number: 1})
          break;
        case 'right':
          this.tiles.push({x: 5, y: 3, number: 1})
          break;
        case 'top':
          this.tiles.push({x: 3, y: 0, number: 1})
          break;
        case 'bottom':
          this.tiles.push({x: 3, y: 5, number: 1})
          break;
        default:

      }
    },

    move: function (event) {
      // console.log('tiles:', this.tiles);
      if (event.key === 'ArrowRight') {
        event.preventDefault()

        this.insertTile('left')

        setTimeout( () => {
          this.moveColumn(3, 'right')
          this.moveColumn(2, 'right')
          this.moveColumn(1, 'right')
          this.moveColumn(0, 'right')
        }, 5)

      } else if (event.key === 'ArrowLeft') {
        event.preventDefault()

        this.insertTile('right')
        setTimeout( () => {
          this.moveColumn(2, 'left')
          this.moveColumn(3, 'left')
          this.moveColumn(4, 'left')
          this.moveColumn(5, 'left')
        }, 5)

      } else if (event.key === 'ArrowUp') {
        event.preventDefault()

        this.insertTile('bottom')
        setTimeout( () => {
          this.moveRow(2, 'up')
          this.moveRow(3, 'up')
          this.moveRow(4, 'up')
          this.moveRow(5, 'up')
        }, 5)

      } else if (event.key === 'ArrowDown') {
        event.preventDefault()

        this.insertTile('top')
        setTimeout( () => {
          this.moveRow(3, 'down')
          this.moveRow(2, 'down')
          this.moveRow(1, 'down')
          this.moveRow(0, 'down')
        }, 5)
      }
    },

    isEmpty: function (x, y) {

      if (this.tiles.find(tile => tile.x === x && tile.y === y)) {
        return false
      } else {
        return true
      }
    },

    combine: function(x, y, number) {
      const targetTile = this.tiles.find(
        tile => {
          if (tile.x === x && tile.y === y) {
            switch (number) {
              case 1:
                if (tile.number === 2) {
                  setTimeout(() => {tile.number = 3}, transitionInterval)
                  return true
                }
                break

              case 2:
                if (tile.number === 1) {
                  setTimeout(() => {tile.number = 3}, transitionInterval)
                  return true
                }
                break
              default:
                setTimeout(() => {targetTile.number = targetTile.number * 2}, transitionInterval)
                return true
            }
          } else {
            return false
          }
        }
      )
      if (targetTile) {
        console.log('targetTile.number:', targetTile.number);
        return true
      } else {
        return false
      }
    },

    getNumberByPosition (x, y) {
      const foundTile = this.tiles.find(tile => tile.x === x && tile.y === y)

      return foundTile.number
    },

    canCombine (num1, num2) {
      switch (num2) {
        case 1:
          if (num1 === num2 && num1 > 2) {
            return true
          }
          break;
        case 2:
          if (num1 + num2 === 3) {
            return true
          }
          break;
          default:
          return false

      }
    },

    moveColumn: function (colNum, direction) {

      const increment = direction === 'right' ? 1 : -1

      const tilesInCol = this.tiles.filter(tile => tile.x === colNum)

      tilesInCol.forEach(tile => {
        const targetX = tile.x + increment
        const targetY = tile.y
        if (this.isEmpty(targetX, targetY)) {
          tile.x = tile.x + increment
        } else if (this.canCombine(tile.number, this.getNumberByPosition(targetX, targetY))){
          this.combine(targetX, targetY, tile.number)
          // TODO: FIX THIS!! this.combine() target? know which is the target and
          //which is sliding onto the target. keep track of that. splice here
        }
      })

      // this.tiles = this.tiles.map((tile, index) => {
      //   if (tile.x === colNum && this.isEmpty(tile.x + increment, tile.y)) {
      //     return Object.assign({}, tile, {x: tile.x + increment})
      //   } else if (tile.x === colNum && this.combine(tile.x + increment, tile.y, tile.number)) {
      //     setTimeout(() => this.tiles.splice(index, 1), transitionInterval)
      //     return Object.assign({}, tile, {
      //       x: tile.x + increment,
      //       // number: tile.number * 2
      //     })
      //   } else {
      //     return tile
      //   }
      // })

    },

    moveRow: function (rowNum, direction) {
      const increment = direction === 'down' ? 1 : -1
      this.tiles = this.tiles.map((tile, index) => {
        if (tile.y === rowNum && this.isEmpty(tile.x, tile.y + increment)) {
          return Object.assign({}, tile, {y: tile.y + increment})
        } else if ( tile.y === rowNum && this.combine(tile.x, tile.y + increment, tile.number)) {
          setTimeout(() => this.tiles.splice(index, 1), transitionInterval)
          return Object.assign({}, tile, {
            y: tile.y + increment,
            // number: tile.number * 2
          })
        } else {
          return tile
        }
      })
    }
    //up and left bottom transformed
    //down and right top transformed


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
</style>
