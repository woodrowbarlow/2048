<template>
  <tile-transition-group class="board" :style="style">
    <tile v-for="tile in tiles"
          :key="tile.key"
          :row="tile.row"
          :col="tile.col"
          :val="tile.val" />
  </tile-transition-group>
</template>

<script>
import Tile from '@/components/Tile.vue'
import TileTransitionGroup from '@/components/TileTransitionGroup.vue'

export default {
  name: 'Board',
  components: {
    Tile,
    TileTransitionGroup,
  },
  props: {
    rows: Number,
    cols: Number,
  },
  data: function() {
    return {
      nextKey: 0,
      tiles: [],
      pendingCreations: [],
      pendingMoves: [],
      pendingMerges: [],
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
    generateKey: function() {
      return this.nextKey++;
    },
    getEmptySpaces: function() {
      var spaces = [];
      var board = this.getBoard();
      for (var r = 0; r < this.rows; r++) {
        for (var c = 0; c < this.cols; c++) {
          if (board[r][c]) {
            continue;
          }
          spaces.append({
            row: r,
            col: c,
          });
        }
      }
      return spaces;
    },
    getTileByKey: function(key) {
      this.tiles.forEach(t => {
        if (t.key == key) {
          return t;
        }
      });
    },
    placeTile: function(r, c, v) {
      this.pendingCreations.append({
        key: this.generateKey(),
        row: r,
        col: c,
        val: v,
      });
    },
    getRow: function(r) {
      var row = [];
      this.tiles.forEach(t => {
        if (t.row == r) {
          row[t.col] = t;
        }
      });
      return row;
    },
    getCol: function(c) {
      var col = [];
      this.tiles.forEach(t => {
        if (t.col == c) {
          col[t.row] = t;
        }
      });
      return col;
    },
    getBoard: function() {
      var board = [];
      this.tiles.forEach(t => {
        if (!board[t.row]) {
          board[t.row] = [];
        }
        board[t.row][t.col] = t;
      });
      return board;
    },
    getPendingRow: function(r) {
      var row = this.getRow(r);
      this.pendingMoves.some(t => {
        if (t.row == r) {
          row.forEach(t2 => {
            if (t2.key == t.key) {
              delete row[t2.col];
              return true;
            }
          });
          row[t.col] = t;
          row[t.col].val = this.getTileByKey(t.key).val;
        }
      });
      this.pendingCreations.forEach(t => {
        if (t.row == r) {
          row[t.col] = t;
        }
      });
      return row;
    },
    getPendingCol: function(c) {
      var col = this.getCol(c);
      this.pendingMoves.some(t => {
        if (t.col == c) {
          col.forEach(t2 => {
            if (t2.key == t.key) {
              delete col[t2.row];
              return true;
            }
          });
          col[t.row] = t;
          col[t.row].val = this.getTileByKey(t.key).val;
        }
      });
      this.pendingCreations.forEach(t => {
        if (t.col == c) {
          col[t.row] = t;
        }
      });
      return col;
    },
    moveTile: function(r, c, dir) {
      var incr = (dir == 'Up' || dir == 'Left') ? -1 : 1;
      var t = null;
      if (dir == 'Up' || dir == 'Down') {
        var col = this.getPendingCol(c);
        t = col[r];
        for (var r2 = r + incr; r2 >= 0 && r2 < this.rows; r2 += incr) {
          if (col[r2] && col[r2].val == t.val) {
            this.pendingMerges.append({
              key1: col[r2].key,
              key2: t.key,
            });
            break;
          }
          else if (col[r2]) {
            r2 -= incr;
            break;
          }
        }
        if (r2 != r) {
          this.pendingMoves.append({
            key: t.key,
            row: r2,
            col: c,
          });
        }
      } else {
        var row = this.getPendingRow(r);
        t = row[c];
        for (var c2 = c + incr; c2 >= 0 && c2 < this.cols; c2 += incr) {
          if (row[c2] && row[c2].val == t.val) {
            this.pendingMerges.append({
              key1: row[c2].key,
              key2: t.key,
            });
            break;
          }
          else if (row[c2]) {
            c2 -= incr;
            break;
          }
        }
        if (c2 != c) {
          this.pendingMoves.append({
            key: t.key,
            row: r,
            col: c2,
          });
        }
      }
    },
    moveRow: function(r, dir) {
      for (var c = 0; c < this.cols; c++) {
        this.moveTile(r, c, dir);
      }
    },
    moveCol: function(c, dir) {
      for (var r = 0; r < this.rows; r++) {
        this.moveTile(r, c, dir);
      }
    },
    move: function(dir) {
      var i = 1;

      switch (dir) {
      case 'Up':
        // skip the top row because it can't move any further
        for (i = 1; i < this.rows; i++) {
          this.moveRow(i, dir);
        }
        break;
      case 'Down':
        // skip the bottom row because it can't move any further
        for (i = this.rows - 2; i >= 0; i--) {
          this.moveRow(i, dir);
        }
        break;
      case 'Left':
        // skip the leftmost column because it can't move any further
        for (i = 1; i < this.cols; i++) {
          this.moveCol(i, dir);
        }
        break;
      case 'Right':
        // skip the rightmost column because it can't move any further
        for (i = this.cols - 2; i >= 0; i--) {
          this.moveCol(i, dir);
        }
        break;
      default:
        return;
      }
    },
    moveTileByKey: function(key, r, c) {
      this.tiles.some(t => {
        if (t.key == key) {
          t.row == r;
          t.col == c;
          return true;
        }
      });
    },
    mergeTilesByKey: function(key1, key2) {
      var t1 = null;
      var t2 = null;
      for (var t = 0; t < this.tiles.length; t++) {
        if (!t1 && this.tiles[t].key == key1) {
          t1 = t;
        }
        if (!t2 && this.tiles[t].key == key2) {
          t2 = t;
        }
      }
      if (t1 && t2 && t1 != t2) {
        this.tiles[t1].val += this.tiles[t2].val;
        this.tiles.splice(t2, 1);
      }
    },
    update: function() {
      this.pendingMoves.forEach(move => {
        this.moveTileByKey(move.key, move.row, move.col);
      });
      this.pendingMerges.forEach(merge => {
        this.mergeTilesByKey(merge.key1, merge.key2);
      });
      this.pendingCreations.forEach(tile => {
        this.tiles.append({
          key: tile.key,
          row: tile.row,
          col: tile.col,
          val: tile.val,
        });
      });
    },
  },
}
</script>

<style scoped lang="scss">
.board {
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
