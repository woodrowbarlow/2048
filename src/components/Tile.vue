<template>
  <div class="tile" :style="style" :data-val="val">
  </div>
</template>

<script>
export default {
  name: 'Tile',
  props: {
    val: Number,
    row: Number,
    col: Number,
  },
  computed: {
    style() {
      // css grids are one-indexed, but javascript arrays are zero-indexed
      return {
        '--row': this.row + 1,
        '--col': this.col + 1,
      };
    },
  },
}
</script>

<style scoped lang="scss">
@keyframes pop-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}

.pop-enter-active {
  animation: pop-in .5s;
}

.pop-leave-active {
  animation: pop-in .5s reverse;
}

.tile {
  display: grid;
  grid-row: var(--row);
  grid-column: var(--col);
  border-radius: 3px;
  font-family: sans-serif;
  font-weight: bold;
}

.tile::before {
  display: block;
  align-self: center;
  justify-self: center;
  content: attr(data-val);
}

.tile[data-val="2"] {
  color: #776e65;
  background: #eee4da;
  font-size: 50px;
}

.tile[data-val="4"] {
  color: #776e65;
  background: #ede0c8;
  font-size: 50px;
}

.tile[data-val="8"] {
  color: #f9f6f2;
  background: #f2b179;
  font-size: 50px;
}

.tile[data-val="16"] {
  color: #f9f6f2;
  background: #f59563;
  font-size: 50px;
}

.tile[data-val="32"] {
  color: #f9f6f2;
  background: #f67c5f;
  font-size: 50px;
}

.tile[data-val="64"] {
  color: #f9f6f2;
  background: #f65e3b;
  font-size: 50px;
}

.tile[data-val="128"] {
  color: #f9f6f2;
  background: #edcf72;
  font-size: 40px;
}

.tile[data-val="256"] {
  color: #f9f6f2;
  background: #edcc61;
  font-size: 40px;
}

.tile[data-val="512"] {
  color: #f9f6f2;
  background: #edc850;
  font-size: 40px;
}

.tile[data-val="1024"] {
  color: #f9f6f2;
  background: #edc53f;
  font-size: 30px;
}

.tile[data-val="2048"] {
  color: #f9f6f2;
  background: #edc22e;
  font-size: 30px;
}
</style>
