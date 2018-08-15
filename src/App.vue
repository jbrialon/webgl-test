<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Shark</router-link> |
      <router-link to="/turtle">Turtle</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
html, body {
  margin:0;
  padding:0;
}
#app {
  position:relative;
  width:100vw;
  height:100vh;
  overflow:hidden;
  background:lightgray;
  text-align:center;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#nav {
  position:absolute;
  top:10px;
  left:90px;
  color:white;
}
#nav a {
  color:white;
}
button {
  border:1px solid black;
  padding: 5px 20px;
  text-transform:uppercase;
  background:none;
  cursor:pointer;
}
</style>
