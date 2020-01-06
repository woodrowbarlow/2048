<template>
  <transition-group name="pop" id="app" tag="div" :style="style">
    <Tile v-for="tile in tiles"
          :key="tile.key"
          :row="tile.row"
          :col="tile.col"
          :val="tile.val" />
  </transition-group>
</template>

<script>
import Tile from './components/Tile.vue'

export default {
  name: 'app',
  components: {
    Tile,
  },
  created() {
    for (var r = 0; r < this.rows; r++) {
      this.board[r] = [];
    }
  },
  mounted() {
    this.spawnTile();
    this.spawnTile();
    window.addEventListener('keydown', (e) => {
      switch (e.code) {
        case 'Space':
          this.spawnTile();
          break;
        case 'ArrowUp':
          break;
        case 'ArrowDown':
          break;
        case 'ArrowLeft':
          break;
        case 'ArrowRight':
          break;
      }
    });
  },
  data: function() {
    return {
      rows: 4,
      cols: 4,
      dropRates: [
        0.3,  // chance of getting a '4'.
        0.1,  // chance of getting an '8'.
        // the next entry would be the chance of getting a '16'.
        // (and so on from there.)
        // anything not in the list has a 0% chance (except '2').
        // any leftover chance is the chance of getting a '2'.
      ],
      board: [],
      tiles: [],
    };
  },
  computed: {
    style() {
      return {
        '--rows': this.rows,
        '--cols': this.cols,
        'grid-template-rows': ' 1fr'.repeat(this.rows),
        'grid-template-columns': ' 1fr'.repeat(this.cols),
      };
    },
  },
  methods: {
    getTiles() {
      var tiles = [];
      for (var r = 0; r < this.rows; r++) {
        if (!this.board[r]) {
          continue;
        }
        for (var c = 0; c < this.cols; c++) {
          if (!this.board[r][c]) {
            continue;
          }
          tiles.push({
            key: this.board[r][c].key,
            row: r,
            col: c,
            val: this.board[r][c].val,
          });
        }
      }
      return tiles;
    },
    getEmptySpaces() {
      var spaces = [];
      for (var r = 0; r < this.rows; r++) {
        for (var c = 0; c < this.cols; c++) {
          if (this.board[r] && this.board[r][c]) {
            continue;
          }
          spaces.push({
            key: 100 * r + c,
            row: r,
            col: c,
          });
        }
      }
      return spaces;
    },
    generateKey: function() {
      var max = 0;
      for (var r = 0; r < this.rows; r++) {
        if (!this.board[r]) {
          continue;
        }
        for (var c = 0; c < this.cols; c++) {
          if (!this.board[r][c]) {
            continue;
          }
          if (this.board[r][c].key > max) {
            max = this.board[r][c].key;
          }
        }
      }
      return max + 1;
    },
    placeTile: function(r, c, v) {
      if (!this.board[r]) {
        this.board[r] = [];
      }
      this.board[r][c] = {
        key: this.generateKey(),
        val: v,
      };
      this.tiles = this.getTiles();
    },
    spawnTile: function() {
      var spaces = this.getEmptySpaces();
      if (!spaces || spaces.length == 0) {
        return;
      }

      var space = spaces[Math.floor(Math.random() * spaces.length)];
      var value = 2;

      var luck = Math.random();
      var drop = 4;
      var bracket = 0.0;

      for (var r = 0; r < this.dropRates.length; r++) {
        var rate = this.dropRates[r];
        if (luck >= bracket && luck < bracket + rate) {
          value = drop;
        }
        bracket += rate;
        drop *= 2;
      }

      this.placeTile(space.row, space.col, value);
    },
  },
}
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

html, body {
  margin: 0;
  padding: 0;
}

body {
  display: grid;
  min-height: 100vh;
  color: #776e65;
  background: #faf8ef;
}

#app {
  align-self: center;
  justify-self: center;
  height: calc(125px * var(--rows));
  width: calc(125px * var(--cols));
  background: #bbada0;
  padding: 15px;
  border-radius: 6px;
  display: grid;
  // why isn't this working?
  // grid-template-rows: repeat(var(--rows), 1fr);
  // grid-template-columns: repear(var(--cols), 1fr);
  grid-gap: 15px;
}
</style>
