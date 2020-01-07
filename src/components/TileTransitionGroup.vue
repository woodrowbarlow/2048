<template>
  <transition-group
      name="tile" duration="500"
      v-bind:css="false"
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter"
      @enter-cancelled="enterCancelled"
      @before-leave="beforeLeave"
      @leave="leave"
      @after-leave="afterLeave"
      @leave-cancelled="leaveCancelled">
    <slot></slot>
  </transition-group>
</template>

<script>
export default {
  name: 'TileTransitionGroup',
  methods: {
    beforeEnter: function(el) {
      el.classList.add('tile-enter');
    },
    enter: function(el) {
      setTimeout(function () {
        el.classList.remove('tile-enter');
        el.classList.add('tile-enter-active');
      }, 250);
    },
    afterEnter: function(el) {
      el.classList.remove('tile-enter-active');
    },
    enterCancelled: function(el) {
      el.classList.remove('tile-enter');
      el.classList.remove('tile-enter-active');
    },
    beforeLeave: function(el) {
      el.classList.add('tile-leave');
    },
    leave: function (el) {
      setTimeout(function () {
        el.classList.remove('tile-leave');
        el.classList.add('tile-leave-active');
      }, 250);
    },
    afterLeave: function(el) {
      el.classList.remove('tile-leave-active');
    },
    leaveCancelled: function(el) {
      el.classList.remove('tile-leave');
      el.classList.remove('tile-leave-active');
    },
  },
}
</script>

<style lang="scss">
.tile-move {
  transition: transform 0.25s;
}

.tile-enter-active {
  transition: transform 0.25s;
}

.tile-leave-active {
  transition: transform 0.25s;
}

.tile-enter,
.tile-leave-active {
  transform: scale(0);
}
</style>
