<template>
  <div id="app">
    <div class="grid">
      <div v-for="n in 6" class="row">
        <div v-for="n in 6" class="cell"></div>
      </div>

      <div v-for="tile in tiles"
            v-bind:style="{top: `${tile.y * 52}px`, left: `${tile.x * 52}px`}"
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
      msg: 'Welcome to Your Vue.js App',
      tiles: [
               {x: 3, y: 3, number: '3'},
               {x: 2, y: 2, number: '2'}
             ]
    }
  },
  methods: {
    move: function (event) {
      if (event.key === 'ArrowRight') {
        event.preventDefault()

        this.tiles = this.tiles.map(tile => {
          if (tile.x < 4) {
            return Object.assign({}, tile, {x: tile.x + 1})
          } else {
            return tile
          }
        })

      } else if (event.key === 'ArrowLeft') {
        event.preventDefault()
        this.tiles = this.tiles.map(tile => {
          if (tile.x > 1) {
            return Object.assign({}, tile, {x: tile.x - 1})
          } else {
            return tile
          }
        })
      } else if (event.key === 'ArrowUp') {
        event.preventDefault()
        this.tiles = this.tiles.map(tile => {
          if (tile.y > 1) {
            return Object.assign({}, tile, {y: tile.y - 1})
          } else {
            return tile
          }
        })
      } else if (event.key === 'ArrowDown') {
        event.preventDefault()
        this.tiles = this.tiles.map(tile => {
          if (tile.y < 4) {
            return Object.assign({}, tile, {y: tile.y + 1})
          } else {
            return tile
          }
        })
      }
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
  border: 1px red solid;
}

.grid {
  position: relative;
}

.tile {
  width: 50px;
  height: 50px;
  border: 1px blue solid;
  position: absolute;
}
</style>
