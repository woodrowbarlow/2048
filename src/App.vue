<template>
  <tile-transition-group id="app" tag="div" :style="style">
    <tile v-for="tile in tiles"
          :key="tile.key"
          :row="tile.row"
          :col="tile.col"
          :val="tile.val" />
  </tile-transition-group>
</template>

<script>
import TileTransitionGroup from './components/TileTransitionGroup.vue'
import Tile from './components/Tile.vue'

export default {
  name: 'app',
  components: {
    TileTransitionGroup,
    Tile,
  },
  mounted() {
    this.spawnTile();
    this.spawnTile();
    window.addEventListener('keydown', (e) => {
      switch (e.code) {
        case 'ArrowUp':
          this.move('Up');
          break;
        case 'ArrowDown':
          this.move('Down');
          break;
        case 'ArrowLeft':
          this.move('Left');
          break;
        case 'ArrowRight':
          this.move('Right');
          break;
      }
    });
  },
  data: function() {
    return {
      rows: 4,
      cols: 4,
      dropRates: [
        0.1, // chance of getting a '4'
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
    moveVertTile: function(r, c, dir) {
      if (!this.board[r] || !this.board[r][c]) {
        return;
      }

      for (var r2 = r; (r2 + dir) >= 0 && (r2 + dir) < this.rows; r2 += dir)
      {
        if (this.board[r2 + dir] && this.board[r2 + dir][c])
        {
          break;
        }
      }

      if (this.board[r2 + dir] && this.board[r2 + dir][c] &&
          this.board[r2 + dir][c].val == this.board[r][c].val)
      {
        this.board[r2 + dir][c].key = this.board[r][c].key;
        this.board[r2 + dir][c].val *= 2;
        this.board[r][c] = null;
        return;
      }

      if (r2 == r) {
        return;
      }

      if (!this.board[r2]) {
        this.board[r2] = [];
      }

      this.board[r2][c] = this.board[r][c];
      this.board[r][c] = null;
    },
    moveVertRow: function(r, dir) {
      if (!this.board[r]) {
        return;
      }

      for (var c = 0; c < this.cols; c++) {
        this.moveVertTile(r, c, dir);
      }
    },
    moveVert: function(dir) {
      var r = 1;
      if (dir < 0) {
        for (r = 1; r < this.rows; r++) {
          this.moveVertRow(r, dir);
        }
      } else if (dir > 0) {
        for (r = this.rows - 2; r >= 0; r--) {
          this.moveVertRow(r, dir);
        }
      }

      this.tiles = this.getTiles();
    },
    moveHorizTile: function(r, c, dir) {
      if (!this.board[r] || !this.board[r][c]) {
        return;
      }

      for (var c2 = c; (c2 + dir) >= 0 && (c2 + dir) < this.cols; c2 += dir)
      {
        if (this.board[r][c2 + dir])
        {
          break;
        }
      }

      if (this.board[r][c2 + dir] &&
          this.board[r][c2 + dir].val == this.board[r][c].val)
      {
        this.board[r][c2 + dir].key = this.board[r][c].key;
        this.board[r][c2 + dir].val *= 2;
        this.board[r][c] = null;
        return;
      }

      if (c2 == c) {
        return;
      }

      this.board[r][c2] = this.board[r][c];
      this.board[r][c] = null;
    },
    moveHorizCol: function(c, dir) {
      for (var r = 0; r < this.rows; r++) {
        if (!this.board[r]) {
          continue;
        }
        this.moveHorizTile(r, c, dir)
      }
    },
    moveHoriz: function(dir) {
      var c = 1;
      if (dir < 0) {
        for (c = 1; c < this.cols; c++) {
          this.moveHorizCol(c, dir);
        }
      } else if (dir > 0) {
        for (c = this.cols - 2; c >= 0; c--) {
          this.moveHorizCol(c, dir);
        }
      }
    },
    move: function(dir) {
      switch(dir) {
        case 'Up':
          this.moveVert(-1);
          break;
        case 'Down':
          this.moveVert(1);
          break;
        case 'Left':
          this.moveHoriz(-1);
          break;
        case 'Right':
          this.moveHoriz(1);
          break;
      }
      this.spawnTile();
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
