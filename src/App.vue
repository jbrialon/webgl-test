<template>
  <div id="app">
    <fbx :play="fbx.play"></fbx>
  </div>
</template>

<script>
import fbx from './components/fbx-viewer.vue'

export default {
  name: 'app',
  data() {
    return {
      fbx: {
        play: false
      }
    }
  },
  components: {
    fbx
  },
  methods: {
    play () {
      // eslint-disable-next-line
      this.fbx.play = !this.fbx.play
    }
  }
}
</script>

<style>
html, body {
  margin:0;
  padding:0;
}
#app {
  width:100vw;
  height:100vh;
  overflow:hidden;
  background:lightgray;
  text-align:center;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

button {
  border:1px solid black;
  padding: 5px 20px;
  text-transform:uppercase;
  background:none;
  cursor:pointer;
}
</style>
