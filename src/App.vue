<template>
  <div id="app">
    <NavBar />
    <transition name="view">
      <router-view
        class="router-view"
        :style="{
          height:
            playList.length > 0 ? 'calc(100% - 2rem)' : 'calc(100% - 1rem)'
        }"
      />
    </transition>
    <Player v-show="playList.length > 0" />
  </div>
</template>

<script>
import NavBar from "./layout/NavBar";
import Player from "./layout/Player";
export default {
  components: { Player, NavBar },
  computed: {
    playList() {
      return this.$store.state.playList.playList;
    }
  }
};
</script>
<style lang="scss">
@import "style/base";
.router-view {
  width: 100%;
  //height: calc(100% - 2rem);
  overflow: hidden;
}
.view-enter,
.view-leave-to {
  opacity: 0;
}
.view-leave-to {
  display: none;
}
.view-enter-active,
.view-leave-active {
  transition: opacity 0.5s;
}
.view-enter-to,
.view-leave {
  opacity: 1;
}
</style>
