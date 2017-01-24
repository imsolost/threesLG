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
export default {
  name: 'app',
  created: function () {
    window.addEventListener('keydown', this.move)
  },
  data: function () {
    return {
      tiles: [
               {x: 1, y: 1, number: 2},
               {x: 2, y: 2, number: 3},
               {x: 3, y: 3, number: 3}
             ]
    }
  },
  methods: {
    move: function (event) {
      if (event.key === 'ArrowRight') {
        event.preventDefault()

        this.moveColumn(3, 'right')
        this.moveColumn(2, 'right')
        this.moveColumn(1, 'right')

      } else if (event.key === 'ArrowLeft') {
        event.preventDefault()

        this.moveColumn(2, 'left')
        this.moveColumn(3, 'left')
        this.moveColumn(4, 'left')

      } else if (event.key === 'ArrowUp') {
        event.preventDefault()

        this.moveRow(2, 'up')
        this.moveRow(3, 'up')
        this.moveRow(4, 'up')

      } else if (event.key === 'ArrowDown') {
        event.preventDefault()

        this.moveRow(3, 'down')
        this.moveRow(2, 'down')
        this.moveRow(1, 'down')

      }
    },

    isEmpty: function (x, y) {

      if (this.tiles.find(tile => tile.x === x && tile.y === y)) {
        return false
      } else {
        return true
      }
    },

    isSameNumber: function(x, y, number) {
      if (this.tiles.find(
        tile => tile.x === x && tile.y === y && tile.number === number
      )) {
        return true
      } else {
        return false
      }
    },

    moveColumn: function (colNum, direction) {
      const increment = direction === 'right' ? 1 : -1
      this.tiles = this.tiles.map(tile => {
        if (tile.x === colNum && this.isEmpty(tile.x + increment, tile.y)) {
          return Object.assign({}, tile, {x: tile.x + increment})
        } else if (tile.x === colNum && this.isSameNumber(tile.x + increment, tile.y, tile.number)) {
          return Object.assign({}, tile, {
            x: tile.x + increment,
            number: tile.number * 2
          })
        } else {
          return tile
        }
      })
    },

    moveRow: function (rowNum, direction) {
      const increment = direction === 'down' ? 1 : -1
      this.tiles = this.tiles.map(tile => {
        if (tile.y === rowNum && this.isEmpty(tile.x, tile.y + increment)) {
          return Object.assign({}, tile, {y: tile.y + increment})
        } else {
          return tile
        }
      })
    }
  }
}
</script>

<style>
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
  box-shadow: inset 0 0 10px black;
  position: absolute;
  text-align: center;
  font-size: 25px;
  font-style: bold;
  line-height: 50px;
  background-color: grey;
  transition: top 0.5s, left 0.5s;
}
</style>
