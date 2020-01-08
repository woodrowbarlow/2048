<template>
  <div id="app">
    <board ref="board" rows="4" cols="4" />
  </div>
</template>

<script>
import Board from './components/Board.vue'

export default {
  name: 'app',
  components: {
    Board,
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
      dropRates: [
        0.1, // chance of getting a '4'
      ],
    };
  },
  methods: {
    spawnTile: function() {
      var board = this.$refs.board;
      var spaces = board.getEmptySpaces();
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

      board.placeTile(space.row, space.col, value);
      board.update();
    },
    move: function(dir) {
      var board = this.$refs.board;
      board.move(dir);
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
}
</style>
